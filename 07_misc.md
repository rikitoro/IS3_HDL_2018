IS3 マイクロコンピュータ基礎 HDL実習

# 6章 その他

---
## 10進カウンタ

````SystemVerilog
module count10 (
  input   logic       clock,
  output  logic [3:0] count
);

  always_ff @ (posedge clock) begin
    if (count >= 3'd10) begin
      count <= 3'd0;
    end else begin
      count <= count + 1'b1;
    end
  end

endmodule
````

---
## parameter

````SystemVerilog
module register #(parameter WIDTH = 8) (
  input                     clock,
  input   logic [WIDTH-1:0] d,
  output  logic [WIDTH-1:0] q
);

  always_ff @ (posedge clock) begin
    q <= d;
  end
endmodule
````

````SystemVerilog
module top (
  input   logic       clock,
  input   logic [1:0] sw_2bit,
  input   logic [7:0] sw_8bit,
  output  logic [1:0] led_2bit,
  output  logic [7:0] led_8bit
);

  // 2-bit レジスタ
  register #(.WIDTH(2)) register_2bit( // registerモジュールのパラメータを上書き(WIDTH = 2)
    .clock  (clock),
    .d      (sw_2bit),
    .q      (led_2bit)
  );

  // 8-bit レジスタ
  register register_8bit (  // パラメータを指定しないとregisterモジュールで指定された値が使われる(WIDTH = 8)
    .clock  (clock),
    .d      (sw_8bit),
    .q      (led_8bit)
  );

endmodule // Top
````
