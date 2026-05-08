# Synchronous FIFO Design and Verification using Verilog

## Introduction

A synchronous FIFO (First-In-First-Out) memory queue ensures sequential data flow between two systems, maintaining synchronization through a common clock. This document outlines the design specifications and test plan for a FIFO module.


<img width="336" height="124" alt="image" src="https://github.com/user-attachments/assets/f4853282-38c4-463f-9b21-1497b1e4528e" />


## Description

The FIFO memory queue facilitates sequential data writes and reads, ensuring no data underflow or overflow occurs. This design follows synchronous operations using a single clock signal for both reading and writing.

## DESIGN CODE 
```
module Synchronous_FIFO #(
    parameter WIDTH = 8,    // Data width
    parameter DEPTH = 16    // FIFO depth
)(
    input wire clk,
    input wire rst_n,
    input wire [WIDTH-1:0] data_in,
    input wire wr_en,
    input wire rd_en,
    output reg [WIDTH-1:0] data_out,
    output wire full,
    output wire empty
);

// Memory and pointers
reg [WIDTH-1:0] fifo [0:DEPTH-1];
reg [$clog2(DEPTH)-1:0] w_ptr, r_ptr;
reg [3:0] count;

// Reset logic
always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        w_ptr <= 0;
        r_ptr <= 0;
        count <= 0;
        data_out <= 0;
    end else begin
        if (wr_en && !full) begin
            fifo[w_ptr] <= data_in;
            w_ptr <= (w_ptr + 1) % DEPTH;
            count <= count + 1;
        end
        if (rd_en && !empty) begin
            data_out <= fifo[r_ptr];
            r_ptr <= (r_ptr + 1) % DEPTH;
            count <= count - 1;
        end
    end
end

// Status flags
assign full = (count == DEPTH);
assign empty = (count == 0);

endmodule
```

## TESTBENCH CODE
```
module tb_Synchronous_FIFO;

parameter WIDTH = 8;
parameter DEPTH = 16;

reg clk, rst_n, wr_en, rd_en;
reg [WIDTH-1:0] data_in;
wire [WIDTH-1:0] data_out;
wire full, empty;

Synchronous_FIFO #(.WIDTH(WIDTH), .DEPTH(DEPTH)) fifo_inst (
    .clk(clk),
    .rst_n(rst_n),
    .data_in(data_in),
    .wr_en(wr_en),
    .rd_en(rd_en),
    .data_out(data_out),
    .full(full),
    .empty(empty)
);

initial begin
    clk = 0;
    forever #5 clk = ~clk;
end

initial begin
    rst_n = 0; wr_en = 0; rd_en = 0;
    #10 rst_n = 1;
    // Run all test cases
    test_case_write_operation();
    test_case_read_operation();
    test_case_full_condition();
    test_case_empty_condition();
    test_case_single_element();
    test_case_multiple_writes();
    test_case_multiple_reads();
    test_case_wrap_around();
    test_case_simultaneous_read_write();
    test_case_reset();
    test_case_overflow();
    test_case_underflow();
    $finish;
end

task test_case_write_operation();
begin
    data_in = 8'hAA;
    wr_en = 1; #10 wr_en = 0;
    if (empty) $display("Error: FIFO should not be empty.");
end
endtask

task test_case_read_operation();
begin
    rd_en = 1; #10 rd_en = 0;
    if (data_out !== 8'hAA) $display("Error: Data read mismatch.");
end
endtask

task test_case_full_condition();
begin
    repeat (DEPTH) begin
        data_in = $random;
        wr_en = 1; #10 wr_en = 0;
    end
    if (!full) $display("Error: FIFO full flag not set.");
end
endtask

task test_case_empty_condition();
begin
    while (!empty) begin
        rd_en = 1; #10 rd_en = 0;
    end
    if (!empty) $display("Error: FIFO empty flag not set.");
end
endtask

task test_case_single_element();
begin
    data_in = 8'h55;
    wr_en = 1; #10 wr_en = 0;
    rd_en = 1; #10 rd_en = 0;
    if (data_out !== 8'h55) $display("Error: Single element mismatch.");
end
endtask

task test_case_multiple_writes();
begin
    for (int i = 0; i < DEPTH; i++) begin
        data_in = i;
        wr_en = 1; #10 wr_en = 0;
    end
end
endtask

task test_case_multiple_reads();
begin
    for (int i = 0; i < DEPTH; i++) begin
        rd_en = 1; #10 rd_en = 0;
        if (data_out !== i) $display("Error: Multiple reads mismatch at index %0d", i);
    end
end
endtask

task test_case_wrap_around();
begin
    for (int i = 0; i < DEPTH; i++) begin
        data_in = i;
        wr_en = 1; #10 wr_en = 0;
    end
    for (int i = 0; i < 2; i++) begin
        rd_en = 1; #10 rd_en = 0;
    end
    data_in = 8'h77;
    wr_en = 1; #10 wr_en = 0;
    rd_en = 1; #10 rd_en = 0;
    if (data_out !== 8'h77) $display("Error: Wrap-around mismatch.");
end
endtask

task test_case_simultaneous_read_write();
begin
    data_in = 8'h99;
    wr_en = 1; rd_en = 1; #10;
    wr_en = 0; rd_en = 0;
    if (data_out !== 8'h99) $display("Error: Simultaneous read/write failed.");
end
endtask

task test_case_reset();
begin
    data_in = 8'hFF;
    wr_en = 1; #10 wr_en = 0;
    rst_n = 0; #10 rst_n = 1;
    if (!empty) $display("Error: FIFO not empty after reset.");
end
endtask

task test_case_overflow();
begin
    for (int i = 0; i < DEPTH + 2; i++) begin
        data_in = $random;
        wr_en = 1; #10 wr_en = 0;
    end
    if (count > DEPTH) $display("Error: FIFO overflow occurred.");
end
endtask

task test_case_underflow();
begin
    rd_en = 1; #10 rd_en = 0;
    if (data_out !== {WIDTH{1'bx}}) $display("Error: Invalid data on underflow.");
end
endtask

endmodule
```
## SIMULATION OUTPUTS

## 1. test_case_write_operation

<img width="982" height="550" alt="image" src="https://github.com/user-attachments/assets/2f895925-f84b-41d6-b292-48e6be8583c9" />


  - When wr_en = 0, then data_out = xx
  
  - When wr_en=1, then data_out = aa

## 2. test_case_read_operation

<img width="974" height="520" alt="image" src="https://github.com/user-attachments/assets/9fdeb42f-7239-4867-b932-91214c37ddeb" />


- When rd_en = 0,then data_out = 0
- When rd_en = 1,then data_out = aa
- The inference is that the FIFO successfully stored the value aa (from a previous write operation) and correctly retrieved it when requested. The data remained stable and valid.

## 3.test_case_full_condition

<img width="987" height="545" alt="image" src="https://github.com/user-attachments/assets/19a918a5-3a2c-445a-8921-e22dc16b5214" />


- When the wr_en = 1
  
- The simulation shows data being written into the system until it is completely full, proving the buffers capacity works as designed.

## 4.test_case_empty_condition

<img width="1352" height="759" alt="image" src="https://github.com/user-attachments/assets/54f4c4a9-d10c-47f0-b515-46f748ec479d" />


The simulation verifies the correct operation of the FIFO empty flag. Initially, after reset, the FIFO is empty and the empty signal is HIGH. When data (AA) is written into the FIFO using wr_en, the empty flag becomes LOW, indicating that the FIFO now contains valid data. After enabling rd_en, the same data is successfully read from the FIFO, and once all stored data is removed, the empty flag returns to HIGH. This confirms that the FIFO correctly detects and indicates the empty condition after the last data element is read.

## 5.test_case_single_element

<img width="1145" height="638" alt="image" src="https://github.com/user-attachments/assets/9ccd2f5a-3f7b-4ab2-aad1-aa46cdbf7cd0" />


- When wr_en = 1, data_in = 55
- When rd_en = 1, then the written data 55 is readed.
- So the written and read data is same.
- Then the empty flag which is initially =0 is changed to the 1.
  

## 6.test_case_multiple_writes();

 <img width="1144" height="615" alt="image" src="https://github.com/user-attachments/assets/cc7bc93f-ee10-4a71-beea-342206bea347" />

- When the wr_en = 1
- Then FIFO successfully writes multiple data values continuously when wr_en is HIGH. The empty flag 
- changes from 1 to 0 after data is written, and the full flag remains 0 because the FIFO is not 
- completely filled. Therefore, the multiple write testcase is working correctly.

## 7.test_case_multiple_reads():

<img width="976" height="544" alt="image" src="https://github.com/user-attachments/assets/cabff997-c755-4f91-9d01-79bc4bb91e82" />


- The simulation verifies the correct operation of the synchronous FIFO during write, read, and simultaneous read/write conditions.
- Data is written and read sequentially in FIFO order, confirming proper data transfer functionality.
- The full and empty signals change correctly according to the FIFO status, indicating successful FIFO operation.

## 8.test_case_wrap_around():

<img width="1346" height="717" alt="image" src="https://github.com/user-attachments/assets/dac2338e-542c-430c-9055-e438e77d97d5" />


- Your write pointer reached the 16th slot and didn't "crash" or stop; it rolled over.
- You are reading back the data in the exact order it was put in, even though the pointers have reset.
- Your empty flag correctly toggles back to 1 once you've finished reading all the data, including the "wrapped" values.

## 9.test_case_simultaneous_read_write():

<img width="1359" height="752" alt="image" src="https://github.com/user-attachments/assets/d0810055-15b0-4ef3-b482-b8b9d6bdb2ca" />

- The FIFO successfully performs a simultaneous read and write at 25ns, where data 99 enters as aa is retrieved.

- The status flags full and empty correctly remain low during this operation, proving the internal counter maintains a stable state when data enters and leaves at once.

- This confirms that your dual-port logic handles simultaneous access and pointer increments without timing violations or data clashing.

## 10.test_case_reset():

<img width="1148" height="627" alt="image" src="https://github.com/user-attachments/assets/cf3dd0a0-37d2-46a9-b6bb-2b42b1a16c60" />

- Pointers reset Correctly.
  
- FIFO returns to Empty State.

## 11.test_case_overflow();

<img width="1142" height="643" alt="image" src="https://github.com/user-attachments/assets/3adcba0d-91a4-4d9d-88f1-5b6e279e04ca" />

- Extra writes ignored

- Overflow proection verified

## 12.test_case_underflow();

<img width="981" height="544" alt="image" src="https://github.com/user-attachments/assets/bb7e8b01-fffd-4bac-9724-0e0eb4ba5f88" />


- The empty flag stays at 1 even when you try to read, showing the FIFO knows it has no data left to give.

- The data_out signal does not change to a wrong value, proving the system ignores read commands when the buffer is empty.

- This confirms your logic safely protects the system from "underflow" errors by blocking illegal reads.













