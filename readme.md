This is an effort to Reverse Engineer parts of the FM Towns Marty.  Primary data in this repo will compose of the CRTC, Downscaler ASIC, Drive protocol and other sections of the console


================ Disc Drive ======================


A very hot Topic.  Marty drive uses a parallel interface for commands while the data from CD-ROM sectors is transmitted serially.
The testing procedure including going through Tatsujin-Oh menu and going through test audio to observe behavior.
The Marty does contain an i2s BUS but the i2s doesn't directly go to the DAC.  The 44.1khz LRCK goes to a Fujitsu ASIC which I assume does some additional DSP before outputting to the analog pipeline.

There are several 4-bit Fujitsu microcontrollers responsible for drive arbitration.  One is located on the disc drive and another on the mainboard.
These form the parallel interface for drive commands.
The standard CD-ROM interface pins denoting Subchannel data and clocking are output from the MN6626 technics controller located on the cd drive.

The lidswitch is directly connected to the drive as well but the drive doesn't require it for operation technically.  It just denotes to the CD drive to begin seeking, in which an FM Towns will move it's lens and traverse looking for the first bootable sector in a Towns CD for an OS.


These interfaces are generally "slow" compared to modern microprocessors.  Emulation of a drive wouldn't be too difficult to achieve but it's important to note that there are overall:

1 parallel interface
1 i2s interface
1 two wire serial interface
1 another two? pins for something I haven't figured output
1 CD-I controller interface.  Uncertain how to classify this interface



=================== Downscaler ASIC =================================

Marty is stuck and trapped at outputting 480i video.  However, the Towns internally processes everything natively and only on the output stage of the CRTC that things begin to change.

To better explain this, the Towns computer processes everything internally as RGB555 and outputs all data to a final Downscaler ASIC.  There is a colorspace conversion that occurs inside this part.

The downscaler asic outputs composite video and two channel S-video in YUV422 colorspace.  The reason the CSC occurs is due to the requirement of S-video.
The ASIC takes inputs of a 28.6366Mhz clock which is cleanly an 8x integer of the NTSC subcarrier frequency of 3.579545 Mhz.
It uses this clock internally for all it's downscaling logic.  Unfortunately, this ASIC doesn't output a pixel clock for 480i whatsoever.  It likely may use it internally for timing though.
The S-video is half of the 28.6366Mhz clock, going to the 2 channel video encoder.  So 8 bit Y and UV is present (with embedded sync?) and it could be possible to wire it to an HDMI transmitter that can run YUV422 at 480i.

The downscaler ASIC will always downscale incoming video no matter what.  The test case of using any VING arcade title like Tatsujin-Oh stands true.
Going through the video options and probing points on the Marty mainboard, it's evident towards the left side of the ASIC that HBLANK or HSYNC goes into the ASIC.  I did find some field polarity pins as well coming OUT and going IN.

So in summary:

-The ASIC outputs YUV422 in two 8 bit channels
-It outputs CSYNC, HSYNC, VSYNC, VBLANK, HBLANK, Field start, FIELD (with marked polarity), multiple strobing signals likely meant for internal timing, 
-There is also a 7.15909Mhz clock output from the ASIC but that can't be the pixel clock at 480i.
-It is suspected but not confirmed there is individual Y, U, and V pixel data on the upper left end of the ASIC but I'm uncertain what clock it would follow.


I have attached some data captures using a DSView logic analyzer. You're free to look at the timings yourself.  I don't think a straight up pipeline from 480i in 24 bit resolution is really possible for HDMI.  The digital luma and chroma however would be more promising.

The other ASIC that outputs pixel data to the FM Towns Marty is near the ZIL/ZIP? type RAM near the floppy disc connector.  That ASIC is the true CRTC for frame building.  However, I haven't found an accurate pixel clock or vsync that would make sense.  If given more time I can likely find out how it's done.

==========================  Rear expansion port =========================


The rear expansion port on Marty is never used for really anything.  However my TC Marty came with the printer attachment and I did make a breakout board.  It could be possible this gives direct DMA access which would give fruition to SCSI, RAM or MIDI peripherals.  More studying is required.