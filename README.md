# Mixed-Signal-STM32-Dev-Board

This project follows the  [Mixed-Signal Hardware Design with KiCad](https://fedevel.com/courses/mixed-signal-hardware-design-with-kicad) 4-layer PCB design course instructed by Philip Salmony of the Phil's Lab youtube channel.  

## System Level Description
### Basic Requirements
    - USB powered, single channel, low frequency analysis and generation capabilities
    - Perform frequency analysis and arbitrary signal generation
### Feature specific requirements
    - USB - <500mA current consumption 
    - Low Frequency - 20Hz - 20kHz (audio band)
    - ADC and DAC (or codec) for analysis and generation, single channel 
### Simple Block Diagram  
![SimpleBlockDiagram](https://github.com/JacobParent7/Mixed-Signal-STM32-Dev-Board/assets/105901480/2797847b-17f6-4e3c-a1bc-c3d00f2641d9)
### Refine requirements
    - USB - low bandwidth and transfer requirements
    - Sample at Nyquist limit (40kHz)
    - Choose higher resolution ADC to minimize quantisation error
![datarate](https://github.com/JacobParent7/Mixed-Signal-STM32-Dev-Board/assets/105901480/e2f4f8e3-7ad7-4880-bcec-dd1402deb6ed)
### USB 2.0
    - Full Speed - Up to 12 Mbits/s (this is plenty enough)
    - High Speed - Up to 480 Mbits/s
