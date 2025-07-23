## MuBook
Brian Guan  
  
MuBook is a x86 portable board which is designed around the LattePanda Mu and has the ability to function as a mini PC running either Linux or Windows.  
  
2025-07-12
--- 
Create the basic board, work out some I/O choices.  

USB 3.0 ---- 2 Ports  
USB 2.0 ---- 2 Ports
1 GBE Ethernet  
HDMI Type A  
M.2 M-key (for NVMe)  
M.2 E-key (for WiFi)  
4x SATA?  

2025-07-20
---

Route all of power regulation and the back USB and HDMI circuits:  

<img width="1134" height="598" alt="image" src="https://github.com/user-attachments/assets/d9f44683-523c-4b48-aa4e-11444eb44875" />  

- Did full calculations for the impedance controlled traces (USB3, HDMI)  
- Designed power input to function with 12V or 15V (might change USBC to 12V)  
- Settled on a 170 x 170 mm board (Mini-ITX standard)  
