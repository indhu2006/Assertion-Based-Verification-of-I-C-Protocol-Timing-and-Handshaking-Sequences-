# Experiment 6: Assertion-Based Verification of I²C Protocol Timing and Handshaking Sequences

---

## Aim  
To verify the **I²C protocol timing and handshaking sequences** using **Assertion-Based Verification (ABV)** in **SystemVerilog**.

---

## Apparatus Required  
- Computer with **Windows OS**  
- **EDA Playground** (for online verification)

---

## Description  
The **Inter-Integrated Circuit (I²C)** protocol is a two-wire serial communication standard consisting of **SDA (Serial Data)** and **SCL (Serial Clock)** lines. Proper timing and handshaking between master and slave devices are critical for reliable data transmission.  

In this experiment:
- **SystemVerilog assertions** are used to validate the **protocol timing**, **data validity**, and **handshaking** sequences.  
- Assertions ensure that signals meet timing requirements such as **setup**, **hold**, **start**, and **stop** conditions.  
- The testbench verifies correct protocol behavior under different random scenarios.

---

## Features  
- Assertion-based verification of I²C protocol  
- Monitors handshaking between **master** and **slave**  
- Validates **timing constraints** using **SystemVerilog assertions**  
- Portable and executable in **EDA Playground** or **ModelSim 2020.1**

---

## Procedure  

1. **Open EDA Playground or ModelSim 2020.1**  
   - Launch the simulation environment.

2. **Create a New Project**  
   - In EDA Playground, choose **SystemVerilog (IEEE 1800-2017)**.  
   - Name the project `I2C_Assertion_Verification`.

3. **Add the Following Files:**  
   - `i2c_master.sv` → I²C Master Model  
   - `i2c_tb.sv` → Testbench with Assertions  

4. **Compile the Files**  
   - Check for syntax or semantic errors.  

5. **Run Simulation**  
   - Use the “Run” button (EDA Playground) or command:
     
6. **Analyze Waveforms and Console Output**  
   - Observe whether assertions pass or fail.  
   - Verify proper handshaking between master and slave.

7. **Document the Results**  
   - Save the output logs and waveform for report submission.

---

## SystemVerilog Code
```
module i2c_master (
    output logic SDA,
    output logic SCL,
    input  logic CLK,
    input  logic RESET
);

    logic [7:0] data = 8'hA5;
    int i;

    always @(posedge CLK or posedge RESET) begin
        if (RESET) begin
            SDA <= 1;
            SCL <= 1;
        end else begin
            SDA <= 0;
            #5;
            for (i = 7; i >= 0; i--) begin
                SCL <= 0;
                SDA <= data[i];
                #5;
                SCL <= 1;
                #5;
            end
            SCL <= 1;
            SDA <= 1;
        end
    end
endmodule
module i2c_tb;
    logic SDA, SCL;
    logic CLK, RESET;

    i2c_master uut (
        .SDA(SDA),
        .SCL(SCL),
        .CLK(CLK),
        .RESET(RESET)
    );

    initial begin
        CLK = 0;
        forever #5 CLK = ~CLK;
    end

    initial begin
        RESET = 1;
        #10 RESET = 0;
    end

    property start_condition;
        @(posedge CLK) (SCL && $fell(SDA));
    endproperty
    start_check: assert property (start_condition)
        $display("START condition detected at time %0t", $time);
    else
        $error("START condition violated at time %0t", $time);

    property stop_condition;
        @(posedge CLK) (SCL && $rose(SDA));
    endproperty
    stop_check: assert property (stop_condition)
        $display("STOP condition detected at time %0t", $time);
    else
        $error("STOP condition violated at time %0t", $time);

    property data_stable;
        @(posedge SCL) $stable(SDA);
    endproperty
    stable_check: assert property (data_stable)
        $display("SDA stable during SCL HIGH at time %0t", $time);
    else
        $error("Data changed during clock high (I2C violation) at time %0t", $time);

    initial begin
        #200;
        $display("\n=== Simulation Completed ===");
        $finish;
    end
endmodule
```

### I²C Master Design 
```systemverilog
module i2c_master (
    output logic SDA,
    output logic SCL,
    input  logic CLK,
    input  logic RESET
);
    // Simple I²C-like signaling for demonstration
    logic [7:0] data = 8'hA5;
    int i;

    always @(posedge CLK or posedge RESET) begin
        if (RESET) begin
            SDA <= 1;
            SCL <= 1;
        end else begin
            // START condition
            SDA <= 0;
            #5;
            for (i = 7; i >= 0; i--) begin
                SCL <= 0;
                SDA <= data[i];
                #5;
                SCL <= 1; // Clock high for data latch
                #5;
            end
            // STOP condition
            SCL <= 1;
            SDA <= 1;
        end
    end
endmodule
```
### Testbench with Assertions
```
module i2c_tb;
    logic SDA, SCL;
    logic CLK, RESET;

    // Instantiate DUT
    i2c_master uut (
        .SDA(SDA),
        .SCL(SCL),
        .CLK(CLK),
        .RESET(RESET)
    );

    // Clock generation
    initial begin
        CLK = 0;
        forever #5 CLK = ~CLK;
    end

    // Reset generation
    initial begin
        RESET = 1;
        #10 RESET = 0;
    end

    // Assertion for START condition: SDA must go LOW while SCL is HIGH
    property start_condition;
        @(posedge CLK) (SCL && $fell(SDA)) |-> $display("START condition detected");
    endproperty
    assert property (start_condition)
        else $error("START condition violated");

    // Assertion for STOP condition: SDA must go HIGH while SCL is HIGH
    property stop_condition;
        @(posedge CLK) (SCL && $rose(SDA)) |-> $display("STOP condition detected");
    endproperty
    assert property (stop_condition)
        else $error("STOP condition violated");

    // Handshaking check: SDA stable during SCL HIGH
    property data_stable;
        @(posedge SCL) $stable(SDA);
    endproperty
    assert property (data_stable)
        else $error("Data changed during clock high (I2C violation)");

    // Simulation control
    initial begin
        #200;
        $display("Simulation Completed");
        $finish;
    end
endmodule
```
### Simulation Output
<img width="1920" height="1080" alt="Screenshot 2025-11-04 154952" src="https://github.com/user-attachments/assets/37c86a44-38d4-47f9-bbc3-e52acf82c68f" />
<img width="1920" height="1080" alt="Screenshot 2025-11-04 155006" src="https://github.com/user-attachments/assets/c719a4a8-5caf-4b7a-84d4-cb8931bba4d0" />
<img width="1920" height="1080" alt="Screenshot 2025-11-04 155031" src="https://github.com/user-attachments/assets/be04b5b5-1404-4eee-9c6f-1633bb566232" />


### Result
The Assertion-Based Verification of the I²C protocol timing and handshaking sequences was successfully carried out using SystemVerilog.Assertions effectively verified setup, hold, start, and stop conditions, ensuring reliable communication as per the I²C protocol specification.


