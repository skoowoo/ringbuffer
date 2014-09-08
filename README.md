ringbuffer
==========

ringbuffer是用来替代Go语言的channel，提高海量数据收发的性能。目前只支持一个写。


#####使用方法


	ring := ringbuffer.NewRing(100, 1000)

	// 一个写端
	go func() {
		var wbuf *ringbuffer.Buffer

		for i := 0; i < 10000; i++ {
			wbuf = ring.Write(wbuf, i)
		}
		ring.Stop(wbuf)
	}()

	// 10个读端
	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {
		wg.Add(1)

		go func() {
			defer wg.Done()

			var (
				rbuf *ringbuffer.Buffer
				e    interface{}
			)

			for {
				if e, rbuf = ring.Read(rbuf); rbuf == nil {
					break
				}
				log.Println(e.(int))
			}
		}()
	}

	wg.Wait()
    
