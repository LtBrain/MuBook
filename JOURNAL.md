## MuBook
Brian Guan  
  
MuBook is a x86 portable board which is designed around the LattePanda Mu and has the ability to function as a mini PC running either Linux or Windows.  
  
2025-07-12
--- 
Create the basic board, work out some I/O choices.  
I'm going off of the Mu's lite carrier project schematic file right now, but will modify a pretty good amount of the IO. Several people in the Lattepanda Mu Discord recommended this route to me because it takes a lot of the guesswork out of the design process, especially with how the HSIO will need to be broken out from the SODIMM. I plan to completely redesign the PCB for my usage though.  
  
USB 3.0 ---- 2 Ports  (7/13 - 7/16, had some issues with KiCad diff pairs)  
USB 2.0 ---- 2 Ports  (Front IO not done yet)  
1 GBE Ethernet  
HDMI Type A  (7/15 - 7/16)  
M.2 M-key (for NVMe)  
M.2 E-key (for WiFi)  
4x SATA?  (7/17 - 7/20, worked on finding suitable ICs for doing PCIE to SATA bridge without sacrificing performance)  

Went with Type A USB because it was easier for me to implement compared to USB Type C, even at USB 3.0 speeds.  
HDMI was needed because I don't plan on using a dGPU, rather the iGPU for monitor display  

M.2 slots for boot drive and probably some WiFi in case ethernet doesn't pan out for some reason.

(Time: 4 Hrs)

2025-07-20
---

Route all of power regulation and the back USB and HDMI circuits:  

<img width="1134" height="598" alt="image" src="https://github.com/user-attachments/assets/d9f44683-523c-4b48-aa4e-11444eb44875" />  

- Did full calculations for the impedance controlled traces (USB3, HDMI)  
- Designed power input to function with 12V or 15V (might change USBC to 12V)  
- Settled on a 170 x 170 mm board (Mini-ITX standard)

I settled on using a 4 layer JLC7628, which gave a nominal diff pair trace spacing of .1016 mm and .1704 trace width for 90 ohms. I put this in as a netclass in KiCad, then had to redo all of the previous routing I had done on USB 3.0 ports because the impedance control I had done was pretty off.  
  
Had to sacrifice the data capabilities on the USBC port for USB-PD negotiation for voltage.  

(Time: 6 Hrs)

2025-07-22
---

Finish the RealTek 1 GBE IC routing, including all of its supply and support components:  

<img width="382" height="613" alt="image" src="https://github.com/user-attachments/assets/5ec1afbd-87c4-4810-b6c1-20e9cc96f326" />  

Ethernet routing was a bit of a hassle, the IC needed a crystal and a bunch of support components to design around, went with a 
pretty basic fanout design, but the side that went to the RJ45 was a bit difficult to route, needed substantial amounts of serpentine traces
to fix the 1-2mm of intra-pair skew.  

I chose a RJ45 jack with integrated LEDs and magnetics just to reduce the amount of things I would have to route out and implement high-speed design on.  


Started the M.2 and PCIE device routing today, made a large breakout of the SODIMM connector's high-speed IO to ease routing for M.2 slots

<img width="269" height="641" alt="image" src="https://github.com/user-attachments/assets/e9424e81-8bdd-48a6-9376-cffc4b3abd9d" />  


Also added fan header and full HDMI support component rigging (actually set up the regulators correctly this time) 

To Do for Tomorrow:
- [ ] Do back I/O checks for any faults/DRC
- [ ] Start work on the SATA PCB routing
- [ ] Work out 4x USB 2.0 instead of 2x on the front I/O  

(Time: 4.5 Hrs)

2025-07-23
---

Completed the M.2 E Key and M Key high speed routing, had to shift the slots around so they would fit better (managed to avoid doing PCIE crossover), and routed all of the PCIE clkreq lines for the 2 slots.  

<img width="1023" height="680" alt="image" src="https://github.com/user-attachments/assets/21773240-ebcf-44e2-946b-d4c6777193b2" />  
  
I finally fixed all of the intrapair skew of every single PCIe trace, DRC came back with a lot less errors :) 

Today, I also put the mounting holes in, set for a 2280 and 2230 sized M.2 card for some extra expandability so that both sizes could be loaded onto the board without issue.    
  
In addition, I did some rerouting at the SODIMM connector to more effectively fit the traces that I would have to route (Decided to put the Row 1 traces on L4 with diff. pair vias and fanout the Row 2 traces on L1, was just much easier to implement. For example, routing the CLKREF traces became much easier when I shunted the R1 traces to the lower layer as much as possible, so i was able to fit it in without making the board excessively odd.

**MAJOR CHANGE**  
  
<img width="724" height="593" alt="image" src="https://github.com/user-attachments/assets/5f16b608-35e6-4ce6-9686-6c41e816d82f" />  
  

After looking at my board, I decided the ITX standard was quite wasteful, as there was a massive open space at the front of the board. Because of this, I decided to cut down the size in this dimension, resizing it was as slim as possible while accomodating the dual M.2 slots if a 2280 sized card were to be used. This a reasonable amount of space for the front IO, so the design wouldn't change that much in terms of features. 
  
Also managed to get rid of all of the back I/O DRC faults and somewhat work out the SATA and 4x USB 2.0.  
  
The board is now approximately 170 cm x 140 cm.

(Time: 6 Hrs)

2025-07-25
---

Today, I added 2 USB-C ports at 2.0 speeds for the front IO, using the HSIO 3 and 5 pins because they were the most exposed ones that I could access without having to reroute the entire chunk of diff pairs.  

<img width="985" height="462" alt="image" src="https://github.com/user-attachments/assets/a7a13cda-b499-4b4c-8a7a-43725b849dd4" />  

Had missed a day because I was doing a lot of PSAT practice, but all of the basic, bare minimum IO is finally in place, so the last thing I need to do is do the SATA schematics and PCB design:  
  
<img width="497" height="388" alt="image" src="https://github.com/user-attachments/assets/bb276bca-b48b-49c6-b114-acde52196e91" />  
  
I also wired up the reset and power pushbuttons, and will probably add a SPI flash chip as my secondary BIOS chip just so I can use the SATA bios without the risk of bricking the MU.  

I am also probably going to hand assemble this as JLCPCB assembly + tariffs has made it insanely expensive (more than 300 USD), but the components are only around 30 dollars and I would only need a hotplate with solderpaste.  
  

(Time: 3 Hrs)

2025-07-26
---

Added 2 USB 2.0 ports to the front IO bay, transitioned the HSIO 10 and 11 traces from the Mu into 2 SATA lines. This involves the unfortunate sacrifice of 2 of my PCIe lanes on the slot, so I'm just going to design without this slot because a PCIe 3.0 x2 is not very common, and I might as well just use those 2 lanes for 2 more SATA plugs with the use of an ASM1061 PCIe to SATA bridge IC. However, these are quite rare, so I had to find some on AliExpress, and extract the symbol and footprint from an EasyEDA file.  
  
<img width="845" height="441" alt="image" src="https://github.com/user-attachments/assets/c80dcb5d-0643-4674-ad8b-0ce84f4094c1" />  
Base setup for the ASM1061  
  
I went for the ASM1061 instead of some more common chips like the Marvell ones because it would be vastly easier to design for a 48 pin IC than a 76 pin IC, and much easier to assemble, especially because I plan to assemble this board by myself. The ASM1061 is a 1 lane PCIe to 2x SATA chip, so in total, this design will get 4 SATA ports. It also has an option of adding a SPI flash chip for custom firmware, but I will probaby skip this because I don't plan on using that feature for complexity reasons.  

I also got some advice to reroute my extremely dense breakout of high-speed traces and add return vias to where they switched layers to avoid some pretty nasty impedance issues:  
<img width="381" height="530" alt="image" src="https://github.com/user-attachments/assets/420a9b9d-b1ff-4db3-80a7-8b01cde42e65" />  
  
I had to do this a lot of times because adding the return vias in such a dense area would have cut some traces, and JLC doesn't offer blind vias. Also, KiCAD decided to reset all of my progress on this section of the board a couple times by closing itself out.  

(Time: 5 Hrs)

2025-07-27
---

Did all of the stuff neccessary for the 4x SATA ports! This involved finalizing the the ASM1061 chip support components, and then I had to reroute the SATA ports that branched off of the HSIO because I had misread the pinouts for the data lines in SATA and thought it was + - + - when it was in fact + - - +.  

Redone SATA:  
  

<img width="685" height="460" alt="image" src="https://github.com/user-attachments/assets/3e413c63-f255-411f-b15b-80c7270bdcaa" />  
  
The ASM1061 was similar to a STM32 microcontroller in terms of how I designed around it. It required a 20 MHZ external clock and very similar power decoupling. Here's the schematic:  

<img width="950" height="581" alt="image" src="https://github.com/user-attachments/assets/6afde024-2995-44a9-8faa-5403877e29dc" />  
  
The PCIe was quite easy to implement just because I had scrapped the PCIe connector design, and could just recycle the HSIO/PCIe lane and the REFCLK for the IC. The IC was actually relatively easy to implement, all of the pins were arranged in a way that made routing very very easy, but the main problem I had begun to run into was just avoiding boxing myself in. A secondary thing that was really annoying was just that the datasheets were very hard to come by, but I managed to find one on a random website and follow the pinout to design the schematics. However, this design was way better and less cluttered than the ones I thought would occur, especially with my designs with a more advanced ASM1166 or a Marvell chip.  
  
PCB:  
<img width="600" height="490" alt="image" src="https://github.com/user-attachments/assets/26c42777-8826-47f6-9db6-f526fc565b7c" />  

With both the Mu's integrated SATA and my external PCIe SATA bridge designed, the board now has 4 SATA III ports, but is missing power. For the sake of simplicity, I might just use an external supply for this as I am not too well-versed in designing for these relatively high amperage situations. I also did some analysis of my current power budget, and I am pretty sure that as of right now, the board won't be able to provide enough current to 4 drives running at the same time, and especially not when they all spin up at the same time.  

I did do some research, and there might be a way to fix this issue by just having many regulators to distribute the stress that so much current would put on my system, but I would want to do some more research to make sure the system will maintain stability.  
  
(Time: 7 Hrs)








