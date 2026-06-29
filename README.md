# AXI_Interconnect
//axi_interconnect.v                                                       // Language: Verilog 2001

`resetall
`timescale 1ns / 1ps
`default_nettype none

/*
 * AXI4 interconnect
 */
module axi_interconnect #
(
    // Number of AXI inputs (slave interfaces)
    parameter S_COUNT = 4,
    // Number of AXI outputs (master interfaces)
    parameter M_COUNT = 4,
    // Width of data bus in bits
    parameter DATA_WIDTH = 32,
    // Width of address bus in bits
    parameter ADDR_WIDTH = 32,
    // Width of wstrb (width of data bus in words)
    parameter STRB_WIDTH = (DATA_WIDTH/8),
    // Width of ID signal
    parameter ID_WIDTH = 8,
    // Propagate awuser signal
    parameter AWUSER_ENABLE = 0,
    // Width of awuser signal
    parameter AWUSER_WIDTH = 1,
    // Propagate wuser signal
    parameter WUSER_ENABLE = 0,
    // Width of wuser signal
    parameter WUSER_WIDTH = 1,
    // Propagate buser signal
    parameter BUSER_ENABLE = 0,
    // Width of buser signal
    parameter BUSER_WIDTH = 1,
    // Propagate aruser signal
    parameter ARUSER_ENABLE = 0,
    // Width of aruser signal
    parameter ARUSER_WIDTH = 1,
    // Propagate ruser signal
    parameter RUSER_ENABLE = 0,
    // Width of ruser signal
    parameter RUSER_WIDTH = 1,
    // Propagate ID field
    parameter FORWARD_ID = 0,
    // Number of regions per master interface
    parameter M_REGIONS = 1,
    // Master interface base addresses
    // M_COUNT concatenated fields of M_REGIONS concatenated fields of ADDR_WIDTH bits
    // set to zero for default addressing based on M_ADDR_WIDTH
    parameter M_BASE_ADDR = 0,
    // Master interface address widths
    // M_COUNT concatenated fields of M_REGIONS concatenated fields of 32 bits
    parameter M_ADDR_WIDTH = {M_COUNT{{M_REGIONS{32'd24}}}},
    // Read connections between interfaces
    // M_COUNT concatenated fields of S_COUNT bits
    parameter M_CONNECT_READ = {M_COUNT{{S_COUNT{1'b1}}}},
    // Write connections between interfaces
    // M_COUNT concatenated fields of S_COUNT bits
    parameter M_CONNECT_WRITE = {M_COUNT{{S_COUNT{1'b1}}}},
    // Secure master (fail operations based on awprot/arprot)
    // M_COUNT bits
    parameter M_SECURE = {M_COUNT{1'b0}}
)
(
    input  wire                            clk,
    input  wire                            rst,

    /*
     * AXI slave interfaces
     */
    input  wire [S_COUNT*ID_WIDTH-1:0]     s_axi_awid,
    input  wire [S_COUNT*ADDR_WIDTH-1:0]   s_axi_awaddr,
    input  wire [S_COUNT*8-1:0]            s_axi_awlen,
    input  wire [S_COUNT*3-1:0]            s_axi_awsize,
    input  wire [S_COUNT*2-1:0]            s_axi_awburst,
    input  wire [S_COUNT-1:0]              s_axi_awlock,
    input  wire [S_COUNT*4-1:0]            s_axi_awcache,
    input  wire [S_COUNT*3-1:0]            s_axi_awprot,
    input  wire [S_COUNT*4-1:0]            s_axi_awqos,
    input  wire [S_COUNT*AWUSER_WIDTH-1:0] s_axi_awuser,
    input  wire [S_COUNT-1:0]              s_axi_awvalid,
    output wire [S_COUNT-1:0]              s_axi_awready,
    input  wire [S_COUNT*DATA_WIDTH-1:0]   s_axi_wdata,
    input  wire [S_COUNT*STRB_WIDTH-1:0]   s_axi_wstrb,
    input  wire [S_COUNT-1:0]              s_axi_wlast,
    input  wire [S_COUNT*WUSER_WIDTH-1:0]  s_axi_wuser,
    input  wire [S_COUNT-1:0]              s_axi_wvalid,
    output wire [S_COUNT-1:0]              s_axi_wready,
    output wire [S_COUNT*ID_WIDTH-1:0]     s_axi_bid,
    output wire [S_COUNT*2-1:0]            s_axi_bresp,
    output wire [S_COUNT*BUSER_WIDTH-1:0]  s_axi_buser,
    output wire [S_COUNT-1:0]              s_axi_bvalid,
    input  wire [S_COUNT-1:0]              s_axi_bready,
    input  wire [S_COUNT*ID_WIDTH-1:0]     s_axi_arid,
    input  wire [S_COUNT*ADDR_WIDTH-1:0]   s_axi_araddr,
    input  wire [S_COUNT*8-1:0]            s_axi_arlen,
    input  wire [S_COUNT*3-1:0]            s_axi_arsize,
    input  wire [S_COUNT*2-1:0]            s_axi_arburst,
    input  wire [S_COUNT-1:0]              s_axi_arlock,
    input  wire [S_COUNT*4-1:0]            s_axi_arcache,
    input  wire [S_COUNT*3-1:0]            s_axi_arprot,
    input  wire [S_COUNT*4-1:0]            s_axi_arqos,
    input  wire [S_COUNT*ARUSER_WIDTH-1:0] s_axi_aruser,
    input  wire [S_COUNT-1:0]              s_axi_arvalid,
    output wire [S_COUNT-1:0]              s_axi_arready,
    output wire [S_COUNT*ID_WIDTH-1:0]     s_axi_rid,
    output wire [S_COUNT*DATA_WIDTH-1:0]   s_axi_rdata,
    output wire [S_COUNT*2-1:0]            s_axi_rresp,
    output wire [S_COUNT-1:0]              s_axi_rlast,
    output wire [S_COUNT*RUSER_WIDTH-1:0]  s_axi_ruser,
    output wire [S_COUNT-1:0]              s_axi_rvalid,
    input  wire [S_COUNT-1:0]              s_axi_rready,

    /*
     * AXI master interfaces
     */
    output wire [M_COUNT*ID_WIDTH-1:0]     m_axi_awid,
    output wire [M_COUNT*ADDR_WIDTH-1:0]   m_axi_awaddr,
    output wire [M_COUNT*8-1:0]            m_axi_awlen,
    output wire [M_COUNT*3-1:0]            m_axi_awsize,
    output wire [M_COUNT*2-1:0]            m_axi_awburst,
    output wire [M_COUNT-1:0]              m_axi_awlock,
    output wire [M_COUNT*4-1:0]            m_axi_awcache,
    output wire [M_COUNT*3-1:0]            m_axi_awprot,
    output wire [M_COUNT*4-1:0]            m_axi_awqos,
    output wire [M_COUNT*4-1:0]            m_axi_awregion,
    output wire [M_COUNT*AWUSER_WIDTH-1:0] m_axi_awuser,
    output wire [M_COUNT-1:0]              m_axi_awvalid,
    input  wire [M_COUNT-1:0]              m_axi_awready,
    output wire [M_COUNT*DATA_WIDTH-1:0]   m_axi_wdata,
    output wire [M_COUNT*STRB_WIDTH-1:0]   m_axi_wstrb,
    output wire [M_COUNT-1:0]              m_axi_wlast,
    output wire [M_COUNT*WUSER_WIDTH-1:0]  m_axi_wuser,
    output wire [M_COUNT-1:0]              m_axi_wvalid,
    input  wire [M_COUNT-1:0]              m_axi_wready,
    input  wire [M_COUNT*ID_WIDTH-1:0]     m_axi_bid,
    input  wire [M_COUNT*2-1:0]            m_axi_bresp,
    input  wire [M_COUNT*BUSER_WIDTH-1:0]  m_axi_buser,
    input  wire [M_COUNT-1:0]              m_axi_bvalid,
    output wire [M_COUNT-1:0]              m_axi_bready,
    output wire [M_COUNT*ID_WIDTH-1:0]     m_axi_arid,
    output wire [M_COUNT*ADDR_WIDTH-1:0]   m_axi_araddr,
    output wire [M_COUNT*8-1:0]            m_axi_arlen,
    output wire [M_COUNT*3-1:0]            m_axi_arsize,
    output wire [M_COUNT*2-1:0]            m_axi_arburst,
    output wire [M_COUNT-1:0]              m_axi_arlock,
    output wire [M_COUNT*4-1:0]            m_axi_arcache,
    output wire [M_COUNT*3-1:0]            m_axi_arprot,
    output wire [M_COUNT*4-1:0]            m_axi_arqos,
    output wire [M_COUNT*4-1:0]            m_axi_arregion,
    output wire [M_COUNT*ARUSER_WIDTH-1:0] m_axi_aruser,
    output wire [M_COUNT-1:0]              m_axi_arvalid,
    input  wire [M_COUNT-1:0]              m_axi_arready,
    input  wire [M_COUNT*ID_WIDTH-1:0]     m_axi_rid,
    input  wire [M_COUNT*DATA_WIDTH-1:0]   m_axi_rdata,
    input  wire [M_COUNT*2-1:0]            m_axi_rresp,
    input  wire [M_COUNT-1:0]              m_axi_rlast,
    input  wire [M_COUNT*RUSER_WIDTH-1:0]  m_axi_ruser,
    input  wire [M_COUNT-1:0]              m_axi_rvalid,
    output wire [M_COUNT-1:0]              m_axi_rready
);

parameter CL_S_COUNT = $clog2(S_COUNT);
parameter CL_M_COUNT = $clog2(M_COUNT);

parameter AUSER_WIDTH = AWUSER_WIDTH > ARUSER_WIDTH ? AWUSER_WIDTH : ARUSER_WIDTH;

// default address computation
function [M_COUNT*M_REGIONS*ADDR_WIDTH-1:0] calcBaseAddrs(input [31:0] dummy);
    integer i;
    reg [ADDR_WIDTH-1:0] base;
    reg [ADDR_WIDTH-1:0] width;
    reg [ADDR_WIDTH-1:0] size;
    reg [ADDR_WIDTH-1:0] mask;
    begin
        calcBaseAddrs = {M_COUNT*M_REGIONS*ADDR_WIDTH{1'b0}};
        base = 0;
        for (i = 0; i < M_COUNT*M_REGIONS; i = i + 1) begin
            width = M_ADDR_WIDTH[i*32 +: 32];
            mask = {ADDR_WIDTH{1'b1}} >> (ADDR_WIDTH - width);
            size = mask + 1;
            if (width > 0) begin
                if ((base & mask) != 0) begin
                   base = base + size - (base & mask); // align
                end
                calcBaseAddrs[i * ADDR_WIDTH +: ADDR_WIDTH] = base;
                base = base + size; // increment
            end
        end
    end
endfunction

parameter M_BASE_ADDR_INT = M_BASE_ADDR ? M_BASE_ADDR : calcBaseAddrs(0);

integer i, j;

// check configuration
initial begin
    if (M_REGIONS < 1 || M_REGIONS > 16) begin
        $error("Error: M_REGIONS must be between 1 and 16 (instance %m)");
        $finish;
    end

    for (i = 0; i < M_COUNT*M_REGIONS; i = i + 1) begin
        if (M_ADDR_WIDTH[i*32 +: 32] && (M_ADDR_WIDTH[i*32 +: 32] < 12 || M_ADDR_WIDTH[i*32 +: 32] > ADDR_WIDTH)) begin
            $error("Error: address width out of range (instance %m)");
            $finish;
        end
    end

    $display("Addressing configuration for axi_interconnect instance %m");
    for (i = 0; i < M_COUNT*M_REGIONS; i = i + 1) begin
        if (M_ADDR_WIDTH[i*32 +: 32]) begin
            $display("%2d (%2d): %x / %02d -- %x-%x",
                i/M_REGIONS, i%M_REGIONS,
                M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH],
                M_ADDR_WIDTH[i*32 +: 32],
                M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] & ({ADDR_WIDTH{1'b1}} << M_ADDR_WIDTH[i*32 +: 32]),
                M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] | ({ADDR_WIDTH{1'b1}} >> (ADDR_WIDTH - M_ADDR_WIDTH[i*32 +: 32]))
            );
        end
    end

    for (i = 0; i < M_COUNT*M_REGIONS; i = i + 1) begin
        if ((M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] & (2**M_ADDR_WIDTH[i*32 +: 32]-1)) != 0) begin
            $display("Region not aligned:");
            $display("%2d (%2d): %x / %2d -- %x-%x",
                i/M_REGIONS, i%M_REGIONS,
                M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH],
                M_ADDR_WIDTH[i*32 +: 32],
                M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] & ({ADDR_WIDTH{1'b1}} << M_ADDR_WIDTH[i*32 +: 32]),
                M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] | ({ADDR_WIDTH{1'b1}} >> (ADDR_WIDTH - M_ADDR_WIDTH[i*32 +: 32]))
            );
            $error("Error: address range not aligned (instance %m)");
            $finish;
        end
    end

    for (i = 0; i < M_COUNT*M_REGIONS; i = i + 1) begin
        for (j = i+1; j < M_COUNT*M_REGIONS; j = j + 1) begin
            if (M_ADDR_WIDTH[i*32 +: 32] && M_ADDR_WIDTH[j*32 +: 32]) begin
                if (((M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] & ({ADDR_WIDTH{1'b1}} << M_ADDR_WIDTH[i*32 +: 32])) <= (M_BASE_ADDR_INT[j*ADDR_WIDTH +: ADDR_WIDTH] | ({ADDR_WIDTH{1'b1}} >> (ADDR_WIDTH - M_ADDR_WIDTH[j*32 +: 32]))))
                        && ((M_BASE_ADDR_INT[j*ADDR_WIDTH +: ADDR_WIDTH] & ({ADDR_WIDTH{1'b1}} << M_ADDR_WIDTH[j*32 +: 32])) <= (M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] | ({ADDR_WIDTH{1'b1}} >> (ADDR_WIDTH - M_ADDR_WIDTH[i*32 +: 32]))))) begin
                    $display("Overlapping regions:");
                    $display("%2d (%2d): %x / %2d -- %x-%x",
                        i/M_REGIONS, i%M_REGIONS,
                        M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH],
                        M_ADDR_WIDTH[i*32 +: 32],
                        M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] & ({ADDR_WIDTH{1'b1}} << M_ADDR_WIDTH[i*32 +: 32]),
                        M_BASE_ADDR_INT[i*ADDR_WIDTH +: ADDR_WIDTH] | ({ADDR_WIDTH{1'b1}} >> (ADDR_WIDTH - M_ADDR_WIDTH[i*32 +: 32]))
                    );
                    $display("%2d (%2d): %x / %2d -- %x-%x",
                        j/M_REGIONS, j%M_REGIONS,
                        M_BASE_ADDR_INT[j*ADDR_WIDTH +: ADDR_WIDTH],
                        M_ADDR_WIDTH[j*32 +: 32],
                        M_BASE_ADDR_INT[j*ADDR_WIDTH +: ADDR_WIDTH] & ({ADDR_WIDTH{1'b1}} << M_ADDR_WIDTH[j*32 +: 32]),
                        M_BASE_ADDR_INT[j*ADDR_WIDTH +: ADDR_WIDTH] | ({ADDR_WIDTH{1'b1}} >> (ADDR_WIDTH - M_ADDR_WIDTH[j*32 +: 32]))
                    );
                    $error("Error: address ranges overlap (instance %m)");
                    $finish;
                end
            end
        end
    end
end

localparam [2:0]
    STATE_IDLE = 3'd0,
    STATE_DECODE = 3'd1,
    STATE_WRITE = 3'd2,
    STATE_WRITE_RESP = 3'd3,
    STATE_WRITE_DROP = 3'd4,
    STATE_READ = 3'd5,
    STATE_READ_DROP = 3'd6,
    STATE_WAIT_IDLE = 3'd7;

reg [2:0] state_reg = STATE_IDLE, state_next;

reg match;

reg [CL_M_COUNT-1:0] m_select_reg = 2'd0, m_select_next;
reg [ID_WIDTH-1:0] axi_id_reg = {ID_WIDTH{1'b0}}, axi_id_next;
reg [ADDR_WIDTH-1:0] axi_addr_reg = {ADDR_WIDTH{1'b0}}, axi_addr_next;
reg axi_addr_valid_reg = 1'b0, axi_addr_valid_next;
reg [7:0] axi_len_reg = 8'd0, axi_len_next;
reg [2:0] axi_size_reg = 3'd0, axi_size_next;
reg [1:0] axi_burst_reg = 2'd0, axi_burst_next;
reg axi_lock_reg = 1'b0, axi_lock_next;
reg [3:0] axi_cache_reg = 4'd0, axi_cache_next;
reg [2:0] axi_prot_reg = 3'b000, axi_prot_next;
reg [3:0] axi_qos_reg = 4'd0, axi_qos_next;
reg [3:0] axi_region_reg = 4'd0, axi_region_next;
reg [AUSER_WIDTH-1:0] axi_auser_reg = {AUSER_WIDTH{1'b0}}, axi_auser_next;
reg [1:0] axi_bresp_reg = 2'b00, axi_bresp_next;
reg [BUSER_WIDTH-1:0] axi_buser_reg = {BUSER_WIDTH{1'b0}}, axi_buser_next;

reg [S_COUNT-1:0] s_axi_awready_reg = 0, s_axi_awready_next;
reg [S_COUNT-1:0] s_axi_wready_reg = 0, s_axi_wready_next;
reg [S_COUNT-1:0] s_axi_bvalid_reg = 0, s_axi_bvalid_next;
reg [S_COUNT-1:0] s_axi_arready_reg = 0, s_axi_arready_next;

reg [M_COUNT-1:0] m_axi_awvalid_reg = 0, m_axi_awvalid_next;
reg [M_COUNT-1:0] m_axi_bready_reg = 0, m_axi_bready_next;
reg [M_COUNT-1:0] m_axi_arvalid_reg = 0, m_axi_arvalid_next;
reg [M_COUNT-1:0] m_axi_rready_reg = 0, m_axi_rready_next;

// internal datapath
reg  [ID_WIDTH-1:0]    s_axi_rid_int;
reg  [DATA_WIDTH-1:0]  s_axi_rdata_int;
reg  [1:0]             s_axi_rresp_int;
reg                    s_axi_rlast_int;
reg  [RUSER_WIDTH-1:0] s_axi_ruser_int;
reg                    s_axi_rvalid_int;
reg                    s_axi_rready_int_reg = 1'b0;
wire                   s_axi_rready_int_early;

reg  [DATA_WIDTH-1:0]  m_axi_wdata_int;
reg  [STRB_WIDTH-1:0]  m_axi_wstrb_int;
reg                    m_axi_wlast_int;
reg  [WUSER_WIDTH-1:0] m_axi_wuser_int;
reg                    m_axi_wvalid_int;
reg                    m_axi_wready_int_reg = 1'b0;
wire                   m_axi_wready_int_early;

assign s_axi_awready = s_axi_awready_reg;
assign s_axi_wready = s_axi_wready_reg;
assign s_axi_bid = {S_COUNT{axi_id_reg}};
assign s_axi_bresp = {S_COUNT{axi_bresp_reg}};
assign s_axi_buser = {S_COUNT{BUSER_ENABLE ? axi_buser_reg : {BUSER_WIDTH{1'b0}}}};
assign s_axi_bvalid = s_axi_bvalid_reg;
assign s_axi_arready = s_axi_arready_reg;

assign m_axi_awid = {M_COUNT{FORWARD_ID ? axi_id_reg : {ID_WIDTH{1'b0}}}};
assign m_axi_awaddr = {M_COUNT{axi_addr_reg}};
assign m_axi_awlen = {M_COUNT{axi_len_reg}};
assign m_axi_awsize = {M_COUNT{axi_size_reg}};
assign m_axi_awburst = {M_COUNT{axi_burst_reg}};
assign m_axi_awlock = {M_COUNT{axi_lock_reg}};
assign m_axi_awcache = {M_COUNT{axi_cache_reg}};
assign m_axi_awprot = {M_COUNT{axi_prot_reg}};
assign m_axi_awqos = {M_COUNT{axi_qos_reg}};
assign m_axi_awregion = {M_COUNT{axi_region_reg}};
assign m_axi_awuser = {M_COUNT{AWUSER_ENABLE ? axi_auser_reg[AWUSER_WIDTH-1:0] : {AWUSER_WIDTH{1'b0}}}};
assign m_axi_awvalid = m_axi_awvalid_reg;
assign m_axi_bready = m_axi_bready_reg;
assign m_axi_arid = {M_COUNT{FORWARD_ID ? axi_id_reg : {ID_WIDTH{1'b0}}}};
assign m_axi_araddr = {M_COUNT{axi_addr_reg}};
assign m_axi_arlen = {M_COUNT{axi_len_reg}};
assign m_axi_arsize = {M_COUNT{axi_size_reg}};
assign m_axi_arburst = {M_COUNT{axi_burst_reg}};
assign m_axi_arlock = {M_COUNT{axi_lock_reg}};
assign m_axi_arcache = {M_COUNT{axi_cache_reg}};
assign m_axi_arprot = {M_COUNT{axi_prot_reg}};
assign m_axi_arqos = {M_COUNT{axi_qos_reg}};
assign m_axi_arregion = {M_COUNT{axi_region_reg}};
assign m_axi_aruser = {M_COUNT{ARUSER_ENABLE ? axi_auser_reg[ARUSER_WIDTH-1:0] : {ARUSER_WIDTH{1'b0}}}};
assign m_axi_arvalid = m_axi_arvalid_reg;
assign m_axi_rready = m_axi_rready_reg;

// slave side mux
wire [(CL_S_COUNT > 0 ? CL_S_COUNT-1 : 0):0] s_select;

wire [ID_WIDTH-1:0]     current_s_axi_awid      = s_axi_awid[s_select*ID_WIDTH +: ID_WIDTH];
wire [ADDR_WIDTH-1:0]   current_s_axi_awaddr    = s_axi_awaddr[s_select*ADDR_WIDTH +: ADDR_WIDTH];
wire [7:0]              current_s_axi_awlen     = s_axi_awlen[s_select*8 +: 8];
wire [2:0]              current_s_axi_awsize    = s_axi_awsize[s_select*3 +: 3];
wire [1:0]              current_s_axi_awburst   = s_axi_awburst[s_select*2 +: 2];
wire                    current_s_axi_awlock    = s_axi_awlock[s_select];
wire [3:0]              current_s_axi_awcache   = s_axi_awcache[s_select*4 +: 4];
wire [2:0]              current_s_axi_awprot    = s_axi_awprot[s_select*3 +: 3];
wire [3:0]              current_s_axi_awqos     = s_axi_awqos[s_select*4 +: 4];
wire [AWUSER_WIDTH-1:0] current_s_axi_awuser    = s_axi_awuser[s_select*AWUSER_WIDTH +: AWUSER_WIDTH];
wire                    current_s_axi_awvalid   = s_axi_awvalid[s_select];
wire                    current_s_axi_awready   = s_axi_awready[s_select];
wire [DATA_WIDTH-1:0]   current_s_axi_wdata     = s_axi_wdata[s_select*DATA_WIDTH +: DATA_WIDTH];
wire [STRB_WIDTH-1:0]   current_s_axi_wstrb     = s_axi_wstrb[s_select*STRB_WIDTH +: STRB_WIDTH];
wire                    current_s_axi_wlast     = s_axi_wlast[s_select];
wire [WUSER_WIDTH-1:0]  current_s_axi_wuser     = s_axi_wuser[s_select*WUSER_WIDTH +: WUSER_WIDTH];
wire                    current_s_axi_wvalid    = s_axi_wvalid[s_select];
wire                    current_s_axi_wready    = s_axi_wready[s_select];
wire [ID_WIDTH-1:0]     current_s_axi_bid       = s_axi_bid[s_select*ID_WIDTH +: ID_WIDTH];
wire [1:0]              current_s_axi_bresp     = s_axi_bresp[s_select*2 +: 2];
wire [BUSER_WIDTH-1:0]  current_s_axi_buser     = s_axi_buser[s_select*BUSER_WIDTH +: BUSER_WIDTH];
wire                    current_s_axi_bvalid    = s_axi_bvalid[s_select];
wire                    current_s_axi_bready    = s_axi_bready[s_select];
wire [ID_WIDTH-1:0]     current_s_axi_arid      = s_axi_arid[s_select*ID_WIDTH +: ID_WIDTH];
wire [ADDR_WIDTH-1:0]   current_s_axi_araddr    = s_axi_araddr[s_select*ADDR_WIDTH +: ADDR_WIDTH];
wire [7:0]              current_s_axi_arlen     = s_axi_arlen[s_select*8 +: 8];
wire [2:0]              current_s_axi_arsize    = s_axi_arsize[s_select*3 +: 3];
wire [1:0]              current_s_axi_arburst   = s_axi_arburst[s_select*2 +: 2];
wire                    current_s_axi_arlock    = s_axi_arlock[s_select];
wire [3:0]              current_s_axi_arcache   = s_axi_arcache[s_select*4 +: 4];
wire [2:0]              current_s_axi_arprot    = s_axi_arprot[s_select*3 +: 3];
wire [3:0]              current_s_axi_arqos     = s_axi_arqos[s_select*4 +: 4];
wire [ARUSER_WIDTH-1:0] current_s_axi_aruser    = s_axi_aruser[s_select*ARUSER_WIDTH +: ARUSER_WIDTH];
wire                    current_s_axi_arvalid   = s_axi_arvalid[s_select];
wire                    current_s_axi_arready   = s_axi_arready[s_select];
wire [ID_WIDTH-1:0]     current_s_axi_rid       = s_axi_rid[s_select*ID_WIDTH +: ID_WIDTH];
wire [DATA_WIDTH-1:0]   current_s_axi_rdata     = s_axi_rdata[s_select*DATA_WIDTH +: DATA_WIDTH];
wire [1:0]              current_s_axi_rresp     = s_axi_rresp[s_select*2 +: 2];
wire                    current_s_axi_rlast     = s_axi_rlast[s_select];
wire [RUSER_WIDTH-1:0]  current_s_axi_ruser     = s_axi_ruser[s_select*RUSER_WIDTH +: RUSER_WIDTH];
wire                    current_s_axi_rvalid    = s_axi_rvalid[s_select];
wire                    current_s_axi_rready    = s_axi_rready[s_select];

// master side mux
wire [ID_WIDTH-1:0]     current_m_axi_awid      = m_axi_awid[m_select_reg*ID_WIDTH +: ID_WIDTH];
wire [ADDR_WIDTH-1:0]   current_m_axi_awaddr    = m_axi_awaddr[m_select_reg*ADDR_WIDTH +: ADDR_WIDTH];
wir
