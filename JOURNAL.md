## MuBook
Brian Guan  
  
MuBook is a x86 portable board which is designed around the LattePanda Mu and has the ability to function as a mini PC running either Linux or Windows.  
  
2025-07-12
--- 
Create the basic board, work out some I/O choices.  

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
