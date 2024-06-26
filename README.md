# Mixed-Signal-STM32-Dev-Board

This project follows the  [Mixed-Signal Hardware Design with KiCad](https://fedevel.com/courses/mixed-signal-hardware-design-with-kicad) 4-layer PCB design course instructed by Philip Salmony of the Phil's Lab youtube channel.  

## System Level Description
### Basic Requirements
    - USB powered, single channel, low frequency analysis and generation capabilities
    - Perform frequency analysis and arbitrary signal generation
### Feature Specific Requirements
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
        - Cortex M0 should be adequate 
        - Avoid BGA or QFN packages (too expensive and benefits not necessary for our use case) 
        - I know STM32 platform well
### PSU
    - Digital Circuitry Supply
        - Switch mode supply is noisy but more efficient than LDO. Works for resillient digital circuitry
    - Analog Circuitry Supply
        - LDO for low noise
        - Less current required compared to digital circuitry typically
    - Power from USB
        - Must regulate due to noise
        - +5V nominal, can drop to +4.5V
        - 500mA can be negotiated, but 150mA is default so design for that

### Simple Converter Theory
    - Higher bits gives better signal to noise ratio
    - Higher sampling rate provides less aliasing with a higher Nyquist limit, however this produces more data to store, process, and stream
    - Consider using oversampling and antialiasing filters on the convert ICs themselves
    - Interface with SPI since data rate is low (typical for microcontrollers)
    
### Analog Circuitry for Converters
    - ADC
        - High-impedance (don't attenuate original signal), ESD protection, RF filtering
        - Antialiasing filter at 1/2 the sampling frequency (Nyquist Rate)
        - Single ended vs differential compatibility
    - DAC
        - Low-impedance with output buffer for driving a load
        - Antialiasing filter at 1/2 the sampling frequency (Nyquist Rate) 
        - Single ended vs differential compatibility

### Miscellaneous Requirements
    - Mechanical Constraints
        - PCB dimensions
        - Mounting Holes
        - Connector Placement
        - Conductive Case?
    - Connectors
        - Debugging interface
        - IO types (SMA, BNC)
    - Peripherals
        - LEDs?
    - Other
        - Timing
        - ESD protection
        - EMC filtering
        - Biasing for analog circuitry 
### Final Block Diagram
![finalblockdiagram](https://github.com/JacobParent7/Mixed-Signal-STM32-Dev-Board/assets/105901480/48f4dbe4-663c-48d4-8cd2-4369597c9597)

## Schematic Design - Power
![1](https://github.com/JacobParent7/Mixed-Signal-STM32-Dev-Board/assets/105901480/603f80e3-1b72-454e-8f95-f3b2b249c0d8)

### Input Pi Filter
    - The input RLC Pi filter from the USB-C has an inductor with a DC resistance of 0.15 Ohms and max DC of 0.5A
    - http://sim.okawa-denshi.jp/en/Fkeisan.htm -> Tools for filter design 
    - Using the website, we can see the Pi Filter has a cutoff in the 1.6 MHz range. For PSU, we want to keep the cutoff high to dampen high frequency noise
    - Be careful; IC transients are hungry for high frequency current and you don't want to starve them. Cutoff only what is needed to pass EMI tests. 
    
![Screenshot 2024-06-25 211942](https://github.com/JacobParent7/Mixed-Signal-STM32-Dev-Board/assets/105901480/de1b405d-80ce-4dce-917f-c72f35ba7fe9)

### Analog LDO regulator - HT75xx
    - Datasheet says to use at least 10uF for input and output caps. We use 22uF to avoid issues due to tolerances and voltage derating. 
    - Series Input resistor with at least 1/8W power rating creates low-pass filter RC with 22uF input capacitor
    - In general, RC filters have a much shallower roll-off compared to the RLC Pi Filter. They also have significant Johnson noise. 
    - Input low-pass has cutoff at 723 Hz. This is low but okay because it feeds a low freq analog supply. 

### Buck Converter
    - Main external circuitry
        - Input caps
        - Output LC filter - use below formula to calculate inductor value
        - Feedback network (formula straight from datasheet and also in schematic) 
        - Sometimes external FETs and diodes
#### [Choosing Inductor Value]
        - Delta IL is ripple current - try to get 20% - 30% of max output current
        - Design for worst case
            - Larger Vin
            - Low current loads
        - Choosing larger inductor than initially calculated
            - +- 20% typical tolerance
            - Max DC and sat current need to be higher than what we require in our worst peak current case (250mA)
            - Low current loads (down to 50mA) will require larger inductors
            - Is shielding important to us?
        - Plotting below function in DESMOS we see lighter loads require much larger inductance (6x) what we calculated, so we chose a 68 uF inductor
        - Although datasheet says we can use 4.7uF minimum caps, we want to consolidate our BOM, so choose 22uF as required by LDO
#### [DNP???]
    - We have a DNP (Do not place) cap because the datasheet recommends it for stability and speed
    - We still want the option to place it, so we will put pads on the PCB for it. 
![Screenshot 2024-06-25 214346](https://github.com/JacobParent7/Mixed-Signal-STM32-Dev-Board/assets/105901480/91e6cef9-52f3-44ba-89f5-1ca48ed08062)

### Bias Generator
    - We have a single supply 0V -> 3.3 V
    - Bias an input to 1/2 VA to give max swing potential for superimposed AC signal 
    - Provide bypass cap as recommended by datasheet (place physically close on board)
    - In high freq designs, the voltage and ground planes seperated by a dielectric board material create a low value high speed capacitor. 

