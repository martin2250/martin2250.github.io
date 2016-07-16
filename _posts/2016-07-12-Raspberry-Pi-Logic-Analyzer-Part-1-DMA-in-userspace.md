---
layout: post
title: Raspberry Pi Logic Analyzer Part 1 - DMA in userspace
bigimg: /img/raspberryPi.jpg
---

I wanted to get my hands on a Logic Analyzer. And, of course, rather than buying one I wanted to build one.

There are many ways to build a simple logic analyzer, here are some:

* use a parallel SRAM chip, add latch, clock, trigger and a way to read it all out
* use USB microcontroller, read all samples in real time
* use a (ideally cheap) microcontroller with lots of memory

I searched around a bit and what I found out was pretty off-putting:

* parallel SRAM usually only comes in < 16Mbit, costs 5€+ per chip and requires a lot of supporting circuitry
* USB microcontrollers that support High-Speed (480Mbit/s) are expensive, come in very small packages (possible to hand solder, but would require a manufactured board) and need programming hardware (unless DFU is available)
* most mid-range Cortex-M controllers have pretty limited RAM and again need a PCB, programmer etc.

But wait! We most of us have a powerful, chep microcontroller with 512MB of RAM laying around somewhere, together with PCB, network interface and all. I'm talking, of course, about the Raspberry Pi, more specifically the original B edition.  
The Raspberry Pi has some pretty good specs that make it apparently perfect for the job:

* 512MB of RAM, enough to hold minutes of samples
* 32 bit wide I/O registers -> possibly 32 channels
* onboard LAN interface, it's galvanically isolated from your PC and you aren't tethered by a short USB cable
* very fast GPIO

Most of you are probably thinking this can't possibly work, the Pi is not meant for real-time, especially with Linux running in the background. Bear with me...

My first attempt was the obvious: simply polling the GPLEV0 (input) register and storing the data in an array. This involves memory-mapping the section of RAM that contains the GPIO register with the rest being pretty straight-forward.  
I didn't really expect it to work properly, and I was right: although the Pi managed a decent average sample rate of around 3MS/s, it had a lot of jitter and of course would be interrupted by the kernel periodically.
The jitter is caused by the Pi's memory bus being used constantly, for loading new instructions and moving around the samples.

###DMA

A much more interestring approach is using the on-board DMA to copy the register into a buffer. The DMA is not subject to interruptions from the kernel, but it still affected by the memory bus jitter.
In free-running mode, it can pull about 10MS/s.
I'm using sigrok to convert my binary sample file to a pulseview file, so I have a nice GUI to view the results:
![pulseview screenshot](/img/pulseview-jitter.png)
Here I'm sampling a 1MHz square wave. The jitter is worse than on this picture, but since I'm writing this about two months later, I don't have any pictures of the actual results here.

#### Anyways, here is the code:
all registers and structs are named exactly after the documentation in the [BCM2835 ARM Peripherals Datasheet](https://www.adafruit.com/images/product-files/2885/BCM2835Datasheet.pdf).
In this userspace version of the code, I used some functions from [Wallacoloo's code](https://github.com/Wallacoloo/Raspberry-Pi-DMA-Example/blob/master/dma-gpio.c). If you want to learn something about the Pi's Peripherals, I definitely recommend checking it out. 

##### memory mapping the peripherals:

	int memfd = open("/dev/mem", O_RDWR | O_SYNC);
	if (memfd == -1) {
		printf("Failed to open /dev/mem (did you remember to run as root?)\n");
		exit(1);
	}

	int pagemapfd = open("/proc/self/pagemap", O_RDONLY);

	volatile uint32_t* gpio, dma;

	gpio = mapPeripheral(memfd, GPIO_BASE);
	dma = mapPeripheral(memfd, DMA_BASE);

##### initializing the DMA

	*(dma + DMAENABLE) |= 1 << 5;		//we're using channel 5 here
	dmaHeader = (struct DmaChannelHeader*)(dma + 0x500/4);

	dmaHeader->CS |= DMA_CS_ABORT;		//make sure to disable dma first.

	dmaHeader->CS |= DMA_CS_RESET;
	dmaHeader->CS |= DMA_CS_END;

##### creating the buffers

	int buffersize = 1000;
	
	uint32_t *bufferCached = makeLockedMem(buffersize);
	
	uint32_t *buffer = makeUncachedMemView(bufferCached, buffersize, memfd, pagemapfd);
	size_t cbPageBytes = sizeof(struct DmaControlBlock);
    void *virtCbPageCached = makeLockedMem(cbPageBytes);
    void *virtCbPage = makeUncachedMemView(virtCbPageCached, cbPageBytes, memfd, pagemapfd);
	
	struct DmaControlBlock *cbArrCached = (struct DmaControlBlock*)virtCbPageCached;
    struct DmaControlBlock *cbArr = (struct DmaControlBlock*)virtCbPage;

When working in userspace, you need to create multiple buffers, one cached for access, one uncached for the DMA. You also need two pages of memory as you need somewhere to store the DMA control blocks as well.

##### creating the control block

	cbArr->SOURCE_AD = GPLEV0_PHY;
	cbArr->DEST_AD = virtToUncachedPhys(bufferCached, pagemapfd);
	cbArr->NEXTCONBK = 0;				//use this for chaining multiple DMA operations
	cbArr->TI = DMA_CB_TI_NO_WIDE_BURSTS | DMA_CB_TI_DEST_INC | 0x010 | 0xF << 12;
	cbArr->TXFR_LEN = buffersize * 4;
	cbArr->NEXTCONBK = 0;

the DMA expects a control block somewhere in memory, which contains it's "instructions"

##### running the DMA

	dmaHeader->CONBLK_AD = virtToUncachedPhys(cbArrCached, pagemapfd);
	dmaHeader->CS = DMA_CS_PRIORITY(7) | DMA_CS_PANIC_PRIORITY(7) | DMA_CS_DISDEBUG; //high priority (max is 7)
    dmaHeader->CS = DMA_CS_PRIORITY(7) | DMA_CS_PANIC_PRIORITY(7) | DMA_CS_DISDEBUG | DMA_CS_ACTIVE; //activate DMA. 

here we tell the DMA where to find it's instructions (control block) and then start it

##### reading the data

	for(int i = 0; i < buffersize; i++)
	{
		printf("0x%x: 0x%x\n", i, buffer[i] & (1<<2));
	}

after the DMA is finished , you can read out the data

####Next Steps
I wasn't satisfied with the performance, especially the jitter. I could make the DMA work slower, so the jitter would be less noticeable, but that seems a bad trade-off.
You could fix it by not having anything running in the backgound, so the bus is idle except for the DMA.  
In an upcoming post, I'll show you how I made this into a complete boot-image that runs without any kernel behind it.