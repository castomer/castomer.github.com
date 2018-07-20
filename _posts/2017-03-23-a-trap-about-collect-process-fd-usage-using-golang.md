---
layout: single
title: "使用golang采集进程的fd导致cpu飘高问题"
categories: golang ops
tags: golang ops


---

监控业务agent使用golang实现的进程监控数据采集，部分线上ha和nginx部分机器10s周期的进程数据采集时agent cpu消耗周期性飘高。

---

第一个版本实现：

```golang

func CalculateFd(pid string) (netFdNum uint, fileFdNum uint, err error) {
	path := "/proc/" + pid + "/fd/"
	
	files, err := ioutil.ReadDir(path)
	if err != nil {
		// log.Errorf("read path:%s ret:%s", path, err)
		return
	}
	
	for _, file := range files {
		readLinkFile, err := os.Stat(path + file.Name())
		if err != nil {
			// log.Errorf("stat file: %s error", path+file.Name())
			continue
		}
		
		if readLinkFile.Mode()&os.ModeSocket != 0 {
			netFdNum++
			continue
		}
		
		if readLinkFile.Mode().IsRegular() {
			fileFdNum++
			continue
		}
	}

	return
}

```

---

经过对比分析发现，agent cpu消耗飘高的机器fd普遍偏高。 `strace`跟踪进去发现在cpu飘高时刻，agent在做fd的统计任务。

初步怀疑因为fd太多（haproxy 32个进程，大概30W是常态吧），所以呢，决定在for循环中sleep 20ms，主动让出cpu。
第二个版本实现：

```golang


func CalculateFd(pid string) (netFdNum uint, fileFdNum uint, err error) {
	path := "/proc/" + pid + "/fd/"

	files, err := ioutil.ReadDir(path)
	if err != nil {
		// log.Errorf("read path:%s ret:%s", path, err)
		return
	}

	cnt := 0
	for _, file := range files {
		cnt++
		if cnt > 1000 {
			time.Sleep(time.Duration(20) * time.Millisecond)
			cnt = 0
		}

		readLinkFile, err := os.Stat(path + file.Name()) // 此处os.Stat也会贡献不少的资源消耗吧
		if err != nil {
			// log.Errorf("stat file: %s error", path+file.Name())
			continue
		}

		if readLinkFile.Mode() & os.ModeSocket != 0 {
			netFdNum++
			continue
		}

		if readLinkFile.Mode().IsRegular() {
			fileFdNum++
			continue
		}
	}

	return
}


```

---

上到测试环境发现，仍然没有解决问题，下一步就考虑暂时不要采集for循环，不要区分file fd和net fd。
一般来说监控对象的业务场景基本决定了用户关注的是file fd和net fd，比较少有两种fd都特别重要的场景。
我们就基于以上假设对代码进行了简化，于是就有了第三个版本实现：


```golang


func CalculateFd(pid string) (fdNum uint, err error) {
	path := "/proc/" + pid + "/fd/"

	files, err := ioutil.ReadDir(path)
	if err != nil {
		// log.Errorf("read path:%s ret:%s", path, err)
		return
	}

	fdNum = uint(len(files))

	return
}


```

---

这个版本的实现上线后，agent的cpu消耗稍有下降，但仍然很高，不符合预期。这都没几行代码了，没的搞了的样子。
然后跳进`ioutil.ReadDir`的实现以后瞬间亮瞎了。。。


```golang

// https://golang.org/src/io/ioutil/ioutil.go#L90

// ReadDir reads the directory named by dirname and returns
// a list of directory entries sorted by filename.
func ReadDir(dirname string) ([]os.FileInfo, error) {
	f, err := os.Open(dirname)
	if err != nil {
		return nil, err
	}
	list, err := f.Readdir(-1)
	f.Close()
	if err != nil {
		return nil, err
	}
	sort.Slice(list, func(i, j int) bool { return list[i].Name() < list[j].Name() })
	return list, nil
}


```

---

在实际开发过程中，之前都是看方法签名满足需求就直接使用了，没仔细看文档。。。它竟然默认自带排序功能，问题的症结就应该是它。
第四个实现版本：


```golang

func CalculateFd(pid string) (fdNum uint, err error) {
	path := "/proc/" + pid + "/fd/"

	file, err := os.Open(path)
	if err != nil || nil == file {

		return
	}
	defer file.Close()

	files, err := file.Readdirnames(0)

	if err != nil {
		return
	}

	fdNum = uint(len(files))

	return
}


```

今天就先写到这里吧。。。

---

# refs
- [golang ioutil.ReadDir实现](https://golang.org/src/io/ioutil/ioutil.go#L90)
- [Russ cox在google groups上对该问题的回复](https://groups.google.com/forum/#!topic/golang-nuts/Q7hYQ9GdX9Q)
- [ioutil.ReadDir官方文档](https://golang.org/pkg/io/ioutil/)
