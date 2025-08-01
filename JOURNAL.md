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

2025-07-28
---

I'm so really close to finishing the PCB. Today I routed the secondary BIOS chip (and had to make a new hierarchal sheet because KiCad keeps crashing on me when I do it elsewhere for some reason). The chip I chose was the WinBond W25Q128, which should have enough capacity to store the LattePanda's SATA bios. Having a secondary BIOS is important in my case because the board will need this SATA bios, but I don't want to risk tampering with the original, on board LattePanda bios, just in case I need to recover the device and not turn it into an expensive paperweight. In this way, I can keep the known, working bios as well as use the SATA features without risk. Changing the bios is accomplished with a pin header I put on the board. Pulling it to ground will select the onboard bios, pulling it high will do the opposite.  
  
<img width="873" height="560" alt="image" src="https://github.com/user-attachments/assets/17a9a3ac-2ac6-431a-8437-2174ee71d86a" />
  
<img width="603" height="475" alt="image" src="https://github.com/user-attachments/assets/50cd94f2-596f-473d-b8e6-c1bebca5f94e" />  
  
Although the schematic was nice and simple, the PCB routing was anything but that. Because the chip uses QSPI, and is relatively critical to the board actually function on boot up, the traces had to be quite close in length. This led to the pretty crazy traces that are there, because I also had to fix crossover of the left side. After I had gotten that sorted out (after like 2 hours of me deleting it and starting over), I settled on the current design. Then I went to do the pin header to control the BIOS, and ended up with a design somewhat like this:  
  
<img width="625" height="493" alt="image" src="https://github.com/user-attachments/assets/1cac220c-48a3-47c7-aaf4-fc686e2b6567" />  
  
After I finished with the BIOS, I could move onto the last pin headers. These are just some pins that carry UART and I2C as well as 3v3 power. This is just nice to have if I were to mount some sensors on this board. The important thing about this part was just that the UART just gave me the ability to debug my board when it breaks with a standard debugging tool.  
  
GPIO Headers:  
<img width="136" height="585" alt="image" src="https://github.com/user-attachments/assets/b2f79454-6f1c-482b-a39a-99fdfe0157d1" />  
<img width="413" height="669" alt="image" src="https://github.com/user-attachments/assets/f93bc326-7626-4691-a407-d6baaca9bd85" />  
  
This is 99 percent of my PCB design done, so I essentially went back in and did some DRC checks. I started with 160 today, but managed to whittle it down to just 18 connection errors. This is mostly attributed to my really bad labeling skills, so I have to go back in to fix those. It did throw some errors that my SATA IC was in the courtyard of the Mu, but I just excluded these violations because the SODIMM gives enough clearance. 
  
<img width="871" height="660" alt="image" src="https://github.com/user-attachments/assets/5ac0dedc-6270-41e5-9f81-eb64b44c2cfa" />

(Time: 5 Hrs)  
  
2025-07-29
---

Today was not particularly fun. On the positive side, my board got reviewed by the folks in the Lattepanda discord, and I was able to fix most of the issues regarding interrupted ground pours and just the overall clutter near my HSIO traces. It started with this: <img width="539" height="329" alt="image" src="https://github.com/user-attachments/assets/68d823cb-ed90-4474-978d-c71c59ba892a" />  
  
And finished with this:  <img width="675" height="564" alt="image" src="https://github.com/user-attachments/assets/a622c479-3171-4b01-a0a3-e507da683fe6" />  

The things that were pointed out in review were focused upon how my VDC plane would have a lot of blocked pathways to reach my power supply pins on the SODIMM, and that was fixed by moving my 5V and 3V3 rails to the outer edge of the PCB so that the central area could carry the VDC power. One of the most annoying issues was that I was forced to route my I2C and UART lines in the internal planes yesterday, but that interrupted a lot of the ground plane under the HSIO traces, which could cause some serious signal loss. To remedy this, I essentially had to shift them to the center of my board, rather than to the side, ending up with a design like this: <img width="359" height="520" alt="image" src="https://github.com/user-attachments/assets/3335d83c-2ef0-4afb-b37f-b8ad7f317b25" />  
  
I had originally started with this mess:  
<img width="1126" height="603" alt="image" src="https://github.com/user-attachments/assets/eccb9c57-cada-4a7d-9946-17e8d8151e5a" />  

The unfortunate thing about having to route it this alternate way is that I would have to drop 2 out of 3 UART headers and a I2C bus. This was definitely worth it in my opinion because it was way too much complication and risk to route these lines without interrupting more important systems on the board, and I wasn't really sure that I would even use these interfaces that much. This routing method allowed me to bypass all of those high-speed lines and only cross over lower frequency data lines (such as the USB 2.0 lines), and just some basic utility pins coming from the M.2 slots. This was a much better deal than the hit or miss situation that I had before.  
  
In addition to that, my USB C port was originally set to receive power with its pull down configuration on the CC pins. This was the opposite of what I wanted, so I had to change the 5.1k value to 22k and pull the pins up so they acted as a power source for any peripheral attached.  
<img width="1030" height="586" alt="image" src="https://github.com/user-attachments/assets/24a4e1df-3de5-45ba-859c-9dc181cf0135" />  

I also split my power layer into 2 sections, VDC for the top section so that the buck converters and Mu could be supplied effectively, and a ground layer below the SODIMM to minimize EMI on the back layer:  
<img width="918" height="612" alt="image" src="https://github.com/user-attachments/assets/e6b763f9-0e36-4b81-8f9e-c74f395bae7f" />  

A final thing I did today was make all of the designators on the PCB clear so that I wouldn't have a very hard time assembling the board by hand, and could easily figure out what part goes where. I did this with the aid of the 3D viewer in KiCad, which displayed my progress so that I didn't have to imagine based on the PCB editor:  

<img width="718" height="454" alt="image" src="https://github.com/user-attachments/assets/8a328193-d845-4c1d-8470-448691b3984f" />  
  
I also began looking at the rough price on JLCPCB, and because my design uses lots of .2 mm vias, I had to go with that more expensive route. This was done because it was just better suited for high speed design and the density of my current traces than .3 mm, and retrofitting .3 mm vias was definitely not an option as that would screw up a lot of my high speed lanes and the ground traces that run alongside.  
  
Special thanks to WifiCable who reviewed my board when it was like 2 AM where they were!  
  
(Time: 7 Hrs)

2025-07-30
---

CAD work and getting the project ready for submission was the primary focus of today, as my PCB was pending a second review (thanks to 0xArya). The case that I'm designing uses honeycomb vents just so it would be easier to print in 1 piece while still providing adequate cooling to the Mu inside. I opted for a rather simple design, especially with the areas around the IO port because I plan to cut my own IO shields out to make the case more seamless. 


<img width="3296" height="2547" alt="maincase" src="https://github.com/user-attachments/assets/b103d9f9-6a37-4e84-9f46-ced15d9cd50c" />  
  
The simplicity of this design makes it a lot easier to design modular parts that could be added later, such as a housing for a 4 bay NAS. The case was designed to use screws which go through the entirity of the case and are secured by nuts, so it would be much easier to mount case parts directly on top of or below the MuBook if I ever wanted to in the future.  

I went with 2 vent areas on the front and the side because that would give me the ability to have an exhaust and intake which draws area directly over the Mu as well as over my power circuitry and Ethernet IC. This would be extremely beneficial, especially to the buck converters as overheating them can cause some pretty bad damage to the chip itself. This design also draws air right over both M.2 slots, cooling any thermally intensive expansions to the board.  
  
Planned airflow:  
<img width="884" height="587" alt="image" src="https://github.com/user-attachments/assets/7fddfa19-ce19-4099-9264-87f4b789051d" />  

Finally, I added a 40mm fan to the side vent to act as an exhaust fan which pulls air throughout the case to cool the entire system. It is purely optional but I think it can definitely benefit the thermals of the board, especially when it is under heavy load.  
  
On the PCB aspect of things, several things were changed from the review. The VDC pour is now a lot more streamlined and avoids a lot of the differential pairs that are routed outwards from the SODIMM connector, and the buck converters have a better flow of electricity. In addition to this, the board has been switched from .4mm/.2mm vias to .4mm/.3mm vias to save approximately 30 dollars + the additional US tariffs. All DRC errors have also been patched and more ground return vias were added to keep the internal ground layers and external ones closer.  
<img width="1214" height="473" alt="image" src="https://github.com/user-attachments/assets/0b137e80-e614-4607-87da-9f77a4d89703" />
  
I also added some nice silkscreen touches to the board just for aesthetics.  

(Time: 5 Hrs)  

2025-07-31
---

Last day! Today was just mostly piecing _everything_ together for submission. The BOM finally got filled out today on Octopart and I exported it to a google sheet and the repo: https://docs.google.com/spreadsheets/d/1PgW1D4j8ihS-p_X-fiT0mnAKiW7zAq-SEehbU5WOdEI/edit?usp=sharing. I finalized the CAD work and the case, finished up the readme and created a production directory to put the gerbers and step files in.  
  
Finished PCB:  
<img width="760" height="642" alt="image" src="https://github.com/user-attachments/assets/35354bfe-3a8a-44ba-9483-cdfd03d1176f" />  

(Time: 8 Hrs)














