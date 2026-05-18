# sv
## Code:
```
module slave_fifo (
    input clk,
    input rst,

    input valid,
    input write_en,
    input read_en,

    input [7:0] data_in,

    output reg [7:0] data_out,
    output reg full,
    output reg empty
);

    // FIFO MEMORY
    reg [7:0] mem [0:3];

    // POINTERS
    reg [1:0] wr_ptr;
    reg [1:0] rd_ptr;
    reg [2:0] count;

    // STATES
    parameter SETUP = 2'b00,
              WRITE = 2'b01,
              READ  = 2'b10;

    reg [1:0] state;

    //-----------------------------------
    // FSM + FIFO
    //-----------------------------------
    always @(posedge clk or posedge rst) begin

        if (rst) begin
            state    <= SETUP;
            wr_ptr   <= 0;
            rd_ptr   <= 0;
            count    <= 0;
            data_out <= 0;
            full     <= 0;
            empty    <= 1;
        end

        else begin

            case(state)

                //--------------------------------
                // SETUP STATE
                //--------------------------------
                SETUP: begin

                    if(valid && write_en)
                        state <= WRITE;

                    else if(valid && read_en)
                        state <= READ;

                end

                //--------------------------------
                // WRITE STATE
                //--------------------------------
                WRITE: begin

                    if(count < 4) begin

                        mem[wr_ptr] <= data_in;

                        wr_ptr <= wr_ptr + 1;
                        count  <= count + 1;

                        $display("WRITE DATA = %0d", data_in);

                    end

                    state <= SETUP;

                end

                //--------------------------------
                // READ STATE
                //--------------------------------
                READ: begin

                    if(count > 0) begin

                        data_out <= mem[rd_ptr];

                        rd_ptr <= rd_ptr + 1;
                        count  <= count - 1;

                        $display("READ DATA = %0d", mem[rd_ptr]);

                    end

                    state <= SETUP;

                end

            endcase

            // FULL CONDITION
            if(count == 4)
                full <= 1;
            else
                full <= 0;

            // EMPTY CONDITION
            if(count == 0)
                empty <= 1;
            else
                empty <= 0;

        end
    end

endmodule

module tb;

    reg clk;
    reg rst;

    reg valid;
    reg write_en;
    reg read_en;

    reg [7:0] data_in;

    wire [7:0] data_out;
    wire full;
    wire empty;

    // DUT
    slave_fifo dut (
        .clk(clk),
        .rst(rst),
        .valid(valid),
        .write_en(write_en),
        .read_en(read_en),
        .data_in(data_in),
        .data_out(data_out),
        .full(full),
        .empty(empty)
    );

    //-----------------------------------
    // CLOCK
    //-----------------------------------
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    //-----------------------------------
    // TEST
    //-----------------------------------
    initial begin

        rst = 1;
        valid = 0;
        write_en = 0;
        read_en = 0;
        data_in = 0;

        #10;
        rst = 0;

        //--------------------------------
        // WRITE
        //--------------------------------
        @(posedge clk);
        valid = 1;
        write_en = 1;
        data_in = 8'd10;

        @(posedge clk);
        data_in = 8'd20;

        @(posedge clk);
        data_in = 8'd30;

        @(posedge clk);
        write_en = 0;

        //--------------------------------
        // READ
        //--------------------------------
        @(posedge clk);
        read_en = 1;

        @(posedge clk);

        @(posedge clk);

        @(posedge clk);
        read_en = 0;

        #20;
        $finish;

    end

    //-----------------------------------
    // MONITOR
    //-----------------------------------
    initial begin
        $monitor("TIME=%0t DATA_IN=%0d DATA_OUT=%0d FULL=%b EMPTY=%b",
                 $time, data_in, data_out, full, empty);
    end

endmodule
```
## Output:

<img width="1919" height="1014" alt="image" src="https://github.com/user-attachments/assets/2b5f54b9-b82e-4c19-bd87-d1ab90b92463" />
