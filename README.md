# zynq7-sdr
This readme contains a description of all repositories, please check out the "help" part to get started. 

This project is about receiving data from the FPGA into the gnuradio environment.
To do so, we wrote a gnuradio compliant source block, some IPs in the FPGA to receive
and control the data flow, and the linux drivers and programs to control the system, making the
gateway between the FPGA and gnuradio. 
Official maintainers are: 

* Guillaume William Bres-Saix <guillaume.bressaix@gmail.com>
* Jean-Michel Friedt <jmfriedt@femto-st.fr>
* Gwenhael Goavec-Merou <gwenhael.goavec-merou@armadeus.com>

###### Things to be improved:

Pass arguments to the linux drivers from the gnuradio environment through variables:
skip the './ram' and './sx1255' part. 

###### A quick presentation
http://www.youtube.com/watch?v=P-XzTaRDNxg&feature=youtu.be   [0:30] is a presentation
of a real-time frequency demodulation of the incoming signal.

#### gnuradio
- - -
This is our gnuradio source block.
The **fpga-src** sub folder contains our gnuradio source block.

###### examples 
Contains some python top blocks, in order to demonstrate the use of the fpga-src block.
In fpga\_to\_file.py, we call our source and simply pass the data flow into a logFile named "data400". 
In fpga\_qt.py, we pass the dataflow into the FFT block and finally into the graphical QT sink. 
Finally, fpga\_wbfm presents a wide band frequency demodulation of the signal coming from the fpga.


To install the source block, please check the "help" section of this readme.


#### src
- - - 
Contains all the project sources.

###### Vivado
Contains valid IPs to use with Vivado (new Xilinx tool).
    
  * sx1255 controls the SX1255(SEMTECH) radiomodem (transceiver), through an AXI-SPI gateway.
  
  * complex\_plps\_if uses 4096x2 axi-lite transfers to send the 
  I/Q samples from the sx1255 IP.
  
  * complex\_dma uses 8192x2 axi-streaming transfers to send the I/Q
  samples through the axi-dma/HP port.
  
  Import the repository to use the IP. The linux driver can be found in the kernel subdirectory, the
  userspace program is found in the userspace subdirectory.

###### xps-edk
  Contains the same valid IPs to use with the XPS/EDK Xilinx tool.
  
#### help
- - -
###### Embedded Linux

Set a work environment for the zedboard-zynq7 by following this page created by P.Ballister --
https://github.com/balister/oe-gnuradio-manifest -- this has been tested on both the zedboard and the zc706.

http://gnuradio.org/redmine/projects/gnuradio/wiki/Zynq has nice informations on out to create the SD-image.

To create a Linux image for the Zybo you could start by following my topic and the included tutorial here, 
[Xilinx Forums - embedded linux on a zybo] (http://forums.xilinx.com/t5/Embedded-Linux/Booting-Linux-on-a-Zybo/td-p/504993).

Create the SD-image and the rootfs needed for Linux to be running on the board. P. Ballister included all the
gnuradio environment to the embedded linux through _open-embedded_.

###### Getting Started
Upload the **gnuradio** directory onto the zynq-board.
Compile our gnuradio source and install the new block on the zynq7 board, to do so: connect to the board then,

```shell 
scp -r gnuradio root@my_zynq:/home/root
ssh root@my_zynq_ip
cd ~/gnuradio/fpga-src-block/fpga-src/
mkdir build_cross
cmake .. 
make 
cd .. 
./Install.sh (expects build_cross as a subdirectory)
```
The fpga-src block is installed and ready to be used in a top.py file.

Create a bitstream using vivado (or XPS), by adding our IPs: SX1255 and RAM - 
connected to each other. 

Program the FPGA to start operating:

```shell
cat bitstream.bin > /dev/xdevcfg
insmod sx1255_board.ko
insmod sx1255_core.ko
echo "programming the radiomodem.."
./sx1255 init [RX_FREQ] [TX_FREQ]
insmod ram_board.ko
insmod ram_core.ko
echo "controlling the application.."
./ram decim 50
./ram start
echo "launching gnuradio ..."
python fpga_qt.py
```

###### FPGA
- - -

You need a stable HDL environment (Vivado or XPS) in order to use our IP-cores (Xilinx Licenses).
Create zynq designs and imports our IP-cores in order to create valid bitstreams.

###### gnuradio
- - -
This project relies on the use of the __open-embedded__ linux distribution made for the zynq7 series.
Following this tutorial, you will get a whole system embedding all of the gnuradio dependencies, needed to
run a gnuradio design flow on the zynq board.


###### Optional
- - -
Our applications/demonstrations involved the use of external usb-sound cards.
The original open embedded kernel only contains ethernet modules/drivers.
We recompile the linux kernel with steps:
```shell
bitbake virtual/kernel (to retrieve the kernel sources)
```
then cross_compile the new sources
```shell
cd ~/oe-repo/tmp-eglibc/work/$MACHINE-oe-linux-gnueabi/linux-xlnx/3.14-xlinx/git
make ARCH=arm xilinx_zynq_defconfig
make ARCH=arm menuconfig  -- add ALSA support & USB OTG
make ARCH=arm CROSS_COMPILE=arm-oe-linux-gnueabi- 
make ARCH=arm CROSS_COMPILE=arm-oe-linux-gnueabi- modules
make ARCH=arm CROSS_COMPILE=arm-oe-linux-gnueabi- INSTALL_MOD_PATH=./mod modules
```
upload the new lib/modules onto the system rootfs.

###### Have FUN!
