# **Whalebox(仿Docker)的爆诞**

---

## 1. **万物起源第一步，先做测试**

### 1.1 **直击心灵的第一问，Namespace是什么？**

命名空间就像一层隔板，有了这层隔板，就会让隔板内的人以为自己独享这片天地，这层隔板有各种各样的，比如`声音隔离`，`视觉隔离`，`嗅觉隔离`等等，暂时可以这么理解。

IPC Namespace 是 Linux Namespace 机制的一部分，专门用于隔离进程间通信资源。它在容器技术中的作用是确保每个容器有自己的 IPC 资源，而不会与宿主机或其他容器共享，从而增强了安全性和资源隔离能力。

而像这样的命名空间还有很多，docker就是采取这些Namespace来实现隔离的。

容器是什么，docker又是什么？

想清楚这一点，我们才能紧接着做一些第一步的测试：隔离

```go
func main() {
	defer func() {
		if r := recover(); r != nil {
			color.Red("Error: %v", r)
			os.Exit(1)
		}
	}()
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	if err := cmd.Run(); err != nil {
		panic(err)
	}
}
```

这里我们意在启动一个新的进程，docker启动的容器有什么特点？隔离，这就是我们要做的第一步，我们通过`Cloneflags`字段来表明我们采取的隔离，这里我们采取了UTS隔离和IPC隔离，能够隔离不同进程通信隔离，比如说输入`ipcmk -Q`创建的消息队列可以在进程间隔离，而UTS可以令子进程随意修改主机名而不影响宿主机，是这一类型的隔离。

如图所示，**hostname隔离**

![image-20250323155812226](./assets/image-20250323155812226.png)

**通信隔离**：

![QQ_1742716718391](./assets/QQ_1742716718391.png)

做完这一步测试之后，虽然一步把所有的隔离都写出来也很不错，但是量太多，这里仅仅是抛砖引玉，为了更简单易懂。

**PID隔离**，`syscall.CLONE_NEWPID`：能够隔离进程间的id，也就是说，id是可以重复的，但是此时如果打印pstree，还是会打印所有的进程树，显然，使用PID隔离是不够的。

**文件隔离**，`syscall.CLONE_NEWNS`：这里能够创建真正的文件隔离的命名空间，为什么pstree会打印所有的进程树？因为在linux里面一切皆文件！但是这里的文件隔离仅仅是创建了一个命名空间，我们还需要真正的通过命令去proc挂载到当前的命名空间上面，如图：

![QQ_1742717877844](./assets/QQ_1742717877844.png)

**User隔离**，`syscall.CLONE_NEWUSER`：如果我们想要在赋予一个用户可以在一个容器内使用root权限，但是呢，又不能让他在宿主机内使用root权限，这个时候，就需要用到User隔离了，当然，此时并不能仅仅是写一个系统调用的参数就结束了，我们还需要设定一个映射，来保证该进程在当前进程是什么权限，在宿主机又持有什么权限：

```go
		UidMappings: []syscall.SysProcIDMap{
			{
				ContainerID: 0,
				HostID:      0,
				Size:        1,
			},
		},
		GidMappings: []syscall.SysProcIDMap{
			{
				ContainerID: 0,
				HostID:      1000,
				Size:        1,
			},
		},
```

在属性中添加上面的参数即可令进程在容器内持有root权限，而不能持有宿主机的root权限，这样就能实现不同进程之间的用户隔离。

**网络隔离**，`syscall.CLONE_NEWNET`：通过这个系统调用，我们可以实现和宿主机的网络隔离，如图所示，新创建的进程里面啥也没有。

![QQ_1742721118550](./assets/QQ_1742721118550.png)

以下是这次测试的完整代码：

```go
package main

import (
	"os"
	"os/exec"
	"syscall"
	"github.com/fatih/color"
)

func main() {
	defer func() {
		if r := recover(); r != nil {
			color.Red("Error: %v", r)
			os.Exit(1)
		}
	}()
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC |
			syscall.CLONE_NEWPID | syscall.CLONE_NEWNS |
			syscall.CLONE_NEWUSER | syscall.CLONE_NEWNET,
		UidMappings: []syscall.SysProcIDMap{
			{
				ContainerID: 0,
				HostID:      0,
				Size:        1,
			},
		},
		GidMappings: []syscall.SysProcIDMap{
			{
				ContainerID: 0,
				HostID:      1000,
				Size:        1,
			},
		},
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	if err := cmd.Run(); err != nil {
		panic(err)
	}
}
```

---

### 1.2 **开天辟地之Cgroups!**

#### (1) **灵魂拷问之Cgroups是啥？**

利用之前的Namespace，我们可以轻易的做到资源隔离，但是docker还需要限制每个容器的空间，不让他们发生争抢，这又是如何做到的？这就需要用到Cgroups了。

**Cgroups组成**

- cgroup：进行进程分组管理，每一个cgroup有一组进程。
- subsystem：一组资源控制的模块。
- hierarchy：将一组cgroup建树，通过树状结构，可以做到继承，例如，cgroup1需要限制cpu使用率，而有一个进程既需要限制cpu，还需要控制I/O，此时可以新建一个cgroup2，并从cgroup1继承对cpu使用率的限制这一特性。

三者调用关系：hierarchy是一棵树，cgroup是一个节点，而subsystem为这棵树赋予属性(限制)，并且一种subsystem只能赋予一棵树上。

cgroup本质上其实是一个文件夹，而docker只是对其的应用，我们可以先直接用bash来试试这个所谓的cgroup

![QQ_1742726368731](./assets/QQ_1742726368731.png)

当我们在该文件夹下面创建cgroup-1和cgroup-2的时候，会将新建的文件夹标记为子cgroup，同时也会自动生成一系列文件：

![QQ_1742726657162](./assets/QQ_1742726657162.png)

**这几个字段是啥？**

- `cgroup.clone_children`：默认为0，如果为1，则会继承父group的配置。
- `cgroup.procs`：包含直接属于当前节点cgroup的所有进程的ID。
- `notify_on_release `和`release_agent`：第一个参数用作标识该cgroup最后一个进程退出时，是否执行了`release_agent`，而第二个参数是一个路径，这俩结合用于清理不再使用的cgroup。
- `tasks`：表示这个cgroup下面的进程ID，如果将一个进程的ID写入其中，便会将这个进程加入这个cgroup。

当我们将当前的终端进程加入cgroup-1的时候：

![QQ_1742727850795](./assets/QQ_1742727850795.png)

可以看见，他已经被添加到cgroup-1了！

那么现在，我们可以**进一步**的，将`subsystem`添加进去。

在这里你可以输入`mount | grep memory`找到对应的路径，我这里是` /sys/fs/cgroup`，然后进入该目录，创建一个`my_cgroup`然后就可以执行接下来的步骤了。

我这里是将stress占用的内存提高到了4GB，不然看不出太大的变化，我才发现stress有多个pid，我这里通过`pgrep -P [启动时的PID]`来查询剩下的stress的pid，不然限制不了，然后将这些pid用`echo [pid] | sudo tee /sys/fs/cgroup/my_cgroup/cgroup.procs`加入你新建的cgroup中间，然后先用`top`观察没限制的时候，再输入`sudo sh -c 'echo 100m > /sys/fs/cgroup/my_cgroup/memory.max'`将内存限制重定向到`memory.max`文件，我跟着书上的内容来，发现还是有挺多地方有变化的。

**限制前**：

![QQ_1742730253380](./assets/QQ_1742730253380.png)

**限制后**：

![QQ_1742730290508](./assets/QQ_1742730290508.png)

这里自己去试一试真的挺有趣的，感觉用vscode的自己加上白色的命令行，像个geek一样，哈哈。

----

#### (2) **随后，便是惊心动魄的Docker是如何应用Cgroups的？**

用过docker的朋友们都知道，我们在`docker run`的时候，往往会携带`-m`的参数，来限制容器所占用的内存大小。

docker会为每个容器在系统中的hierarchy创建cgroup。

哎，貌似是版本不一样，好多地方目录也跟书上的不一样

我这里创建了一个docker容器：

![QQ_1742732598770](./assets/QQ_1742732598770.png)

随后输入`ls /sys/fs/cgroup/system.slice/ | grep docker`

这个时候会出现我们容器的文件夹，格式为`docker-容器id`

在这个文件夹下面，这里我就输入`cd docker-2333f52ef2764680b48e152d9305bb0544fb2f2bc7f1d45d2456caef75f3d152.scope/`

然后ls，就是我们熟悉的页面了

![QQ_1742732779770](./assets/QQ_1742732779770.png)

我们从刚刚的测试中是知道的，memory.max文件中藏着最大内存的限制：

![QQ_1742732804556](./assets/QQ_1742732804556.png)

perfect！测试完成，docker完美的将我们设置的参数赋予了我们创建的cgroup。

---

#### (3) **再加点料，Go中新增Cgroup的限制**

这里我调试了半天，发现stress甚至还会产生很多子进程，如果按照书上的来，很多都跑不通，无敌了，下面是能够正确跑通的代码：

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
	"path"
	"strings"
	"syscall"

	"github.com/fatih/color"
)

const CgroupPath = "/sys/fs/cgroup/"

func GetAllChinldpids(pids []string) []string {
	for i := 0; i < len(pids); i++ {
		children, err := exec.Command("pgrep", "-P", pids[i]).Output()
		if err != nil {
			if exitErr, ok := err.(*exec.ExitError); ok && exitErr.ExitCode() == 1 {
				continue
			}
			color.Red("Error: %v", err)
			os.Exit(14)
		}
		pids = append(pids, strings.Fields(string(children))...)
	}
	return pids
}

func main() {
	if os.Args[0] == "/proc/self/exe" {
		fmt.Println("pid:", os.Getpid())

		cmd := exec.Command("sh", "-c", `stress --vm-bytes 2048m --vm-keep -m 1`)

		cmd.SysProcAttr = &syscall.SysProcAttr{}

		cmd.Stdin = os.Stdin
		cmd.Stdout = os.Stdout
		cmd.Stderr = os.Stderr
		if err := cmd.Run(); err != nil {
			fmt.Println(err)
			os.Exit(11)
		}
	}

	cmd := exec.Command("/proc/self/exe")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS |
			syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	if err := cmd.Start(); err != nil {
		fmt.Println(err)
		os.Exit(13)
	} else {
		pids := GetAllChinldpids([]string{fmt.Sprintf("%d", cmd.Process.Pid)})

		fmt.Println("child pid:", pids)

		fmt.Println("pids:", pids)

		os.Mkdir(path.Join(CgroupPath, "TestMemoryLimit"), 0755)

		for _, pid := range pids {
			os.WriteFile(path.Join(CgroupPath, "TestMemoryLimit", "cgroup.procs"), []byte(pid), 0644)
		}

		os.WriteFile(path.Join(CgroupPath, "TestMemoryLimit", "memory.max"), []byte("100m"), 0644)
	}
	cmd.Process.Wait()
}
```

像这样，我们就能够手动启动一个进程，并且这个进程受到Cgroup的限制了！在bash中输入`go run main.go`，然后启动另一个终端，输入`top`。你会发现内存占用被限制了！

#### (4) **Union File System**

简单来说就是将其他文件系统合并到一个文件系统，被合并的文件叫做分支branch，用户在使用读取操作的时候，尽管底层是读取的同一个文件，但是实际上用户会认为他是在独享一个文件系统，而在用户真正对其进行写操作时，才会真正的开辟一个新的内存空间，来将被修改或者写入的数据(不是全部！)放入这段空间，这就叫**COW(写时复制)**，而在docker中也有类似的实现，所有的容器共享一个基础镜像，这一层叫做**镜像层**，而在用户修改容器数据的时候，就会将这部分数据写入到**可写层**，从而达到节省空间的目的。

**AUFS**，他重写了早期的UnionFS，具有快速启动容器，存储性能高等优点，而docker早期正是采取的这一存储方式。

**实践一下吧！**

这里我选择alpine作为基础镜像，实际上用啥都行，这里就只需要在基础镜像的基础上，echo一段字符到文件里就行了。

首先随便创建一个新的目录，创建dockerfile文件：

```dockerfile
FROM alpine:latest

RUN echo "Ciallo World!" > newfile
```

随后，在这个目录下面输入`docker build -t changed-image .`的指令，然后就可以创建一个自己的镜像了.

然后我们可以通过`docker history changed-image`来查看这个容器的历史记录

![QQ_1742802906012](./assets/QQ_1742802906012.png)

我们发现，我们的我们最上层的image layer仅仅使用了14B的空间，这也证明了AUFS是高效地在使用磁盘空间的。

