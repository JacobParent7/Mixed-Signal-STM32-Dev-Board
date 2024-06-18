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
### USB Standard
    - Full Speed - Up to 12 Mbits/s (this is plenty enough)
    - High Speed - Up to 480 Mbits/s
    - System is bus powered (not a host) and will most likely require <500mA
    - Use type C, new universal standard
### Processor
    - Low bandwidth and storage requirements so FPGA unnecessary
    - Use MCU
        - C easy compared to Verilog
        - SWD or JTAG protocols available 
    - Peripherals
        - USB for data stream
        - SPI for ADC and DAC
    - Required Power?
        - Cortex M0 should be adaquate 
        - Avoid BGA or QFN packages (too expensive and not necessary) 
        - I know STM32 platform well
### PSU
    - Digital Circuitry Supply
        - Switch mode supply is noisy but more efficient than LDO. Works for resillient digital circuitry. 
    - Analog Circuitry Supply
        - LDO for low noise
        - Less current required compared to digital circuitry typically
    - Power from USB
        - Must regulate due to noise
        - +5V nominal, can drop to +4.5V
        - 500mA can be negotiated, but 150mA is default so design for that

### Converters
    - Higher bits gives better signal to noise ratio
    - Higher sampling rate provides less aliasing with a higher Nyquist limit, however this produces more data to store, process, and stream
    - Consider using oversampling and antialiasing filters on the convert ICs themselves
    - Interface with SPI since data rate is low (typical for microcontrollers)
