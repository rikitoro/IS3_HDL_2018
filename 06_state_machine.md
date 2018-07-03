IS3 マイクロコンピュータ基礎 HDL実習

# 6章 状態機械の設計

前章までで、レジスタの設計方法と組み合わせ回路の設計方法、さらにそれらの回路モジュールを組み合わせて新しい回路を設計する方法を学んできました。
レジスタと組み合わせ回路とを組み合わせることで、状態機械を設計することができます。
本章では、状態機械の設計方法を学びます。

---
## 回路仕様



![状態機械 my_stm](./assets/my_stm_circuit.png "状態機械 my_stm")

<図6.1 状態機械 my_stm>


![my_stm の状態遷移図](./assets/my_stm.png "my_stm の状態遷移図")

<図6.2 my_stm の状態遷移図>

---
## 回路構成

![my_stm の内部構成](./assets/my_stm_structure.png "my_stm の内部構造")

<図6.3 my_stm の内部構成>

---
## 状態の符号化

<表6.1 状態の符号化>

| 状態 | 符号 state[1:0] |
|------|-----------------|
| SA | 00 |
| SB | 01 |
| SC | 10 |
| SD | 11 |

<リスト6.1 register モジュール(非同期リセット付き2ビットレジスタ)>

```SystemVerilog
module register ( // 非同期リセット付き2ビットレジスタ
  input   logic       clock,
  input   logic       n_reset, // active low async reset
  input   logic [1:0] d,
  output  logic [1:0] q
);

  always_ff @ (posedge clock, negedge n_reset) begin
    if (n_reset == 1'b0) begin
      q <= 2'b00;  // SA にリセット
    end else begin
      q <= d;
    end
  end

endmodule
```

---

## 次状態関数回路

<表6.2 my_stm の状態遷移表>

| p | 今の状態S(N) | 次の状態S(N+1) |
|---|--------------|----------------|
| 0 | SA | SB |
| 0 | SB | SC |
| 0 | SC | SD |
| 0 | SD | SA |
| 1 | SA | SA |
| 1 | SB | SB |
| 1 | SC | SC |
| 1 | SD | SD |


<表6.3 次状態生成回路 next_state_generatorの真理値表>

| 入力 p | 入力 state[1:0] | 出力 next_state[1:0] |
|---|--------------|----------------|
| 0 | 00 | 01 |
| 0 | 01 | 10 |
| 0 | 10 | 11 |
| 0 | 11 | 00 |
| 1 | 00 | 00 |
| 1 | 01 | 01 |
| 1 | 10 | 10 |
| 1 | 11 | 11 |


<リスト6.2 next_state_generator モジュール(次状態生成回路)>

```SystemVerilog
module next_state_generator (
  input   logic [1:0] state,
  input   logic       p,
  output  logic [1:0] next_state
);

  always_comb begin
    case ({p, state})
      3'b0_00:  next_state = 2'b01;
      3'b0_01:  next_state = 2'b10;
      3'b0_10:  next_state = 2'b11;
      3'b0_11:  next_state = 2'b00;
      3'b1_00:  next_state = 2'b00;
      3'b1_01:  next_state = 2'b01;
      3'b1_10:  next_state = 2'b10;
      3'b1_11:  next_state = 2'b11;
      default:  next_state = 2'b00;
    endcase
  end

endmodule //
```

---

## 出力関数

<表6.4 状態と出力の対応表>

| 今の状態S(N) | 出力 y[1:0] |
|--------------|-------------|
| SA | 00 |
| SB | 01 |
| SC | 00 |
| SD | 10 |


<表6.5 出力関数回路 output_decoder の真理値表>

| 入力 state[1:0] | 出力 y[1:0] |
|-----------------|-------------|
| 00 | 00 |
| 01 | 01 |
| 10 | 00 |
| 11 | 10 |


<リスト6.3 output_decoder モジュール(出力関数回路)>

```SystemVerilog
module output_decoder (
  input   logic [1:0] state,
  output  logic [1:0] y
);

  always_comb begin
    case (state)
      2'b00   :  y = 2'b00;
      2'b01   :  y = 2'b10;
      2'b10   :  y = 2'b00;
      2'b11   :  y = 2'b01;
      default :  
    endcase
  end

endmodule //
```

---

## 状態機械の組み上げ

<リスト6.4 my_stm モジュール>

```SystemVerilog
module my_stm (
  input   logic       clock,
  input   logic       n_reset,
  input   logic       p,
  output  logic [1:0] y
);

  logic [1:0] state;      // 今の状態
  logic [1:0] next_state; // 次の状態

  // 状態レジスタ
  register state_register(
    .clock    (clock),
    .n_reset  (n_reset),
    .d        (next_state),
    .q        (state)
  );

  // 次状態関数
  next_state_generator next_state_generator(
    .p          (p),
    .state      (state),
    .next_state (next_state)
  );

  // 出力関数
  output_decoder output_decoder(
    .state      (state),
    .y          (y)
  );

endmodule //
```

## 演習

リスト6.4 my_stm モジュールを実習ボード DE0-CV に実装してその動作を確認しましょう。
my_stm モジュールの入出力信号は表5.6のように DE0-CV の入出力デバイスに割り当ててください。

なお、リスト6.1 register モジュール、リスト6.2 next_state_generator モジュール、およびリスト6.3 output_decoder モジュールも必要となりますので、プロジェクトにそれらのデザインファイルも追加しましょう。

<表6.6 my_stm モジュールの入出力のデバイスへの割り当て>

|信号名|割り当てデバイス|入出力|
|------|----------------|------|
|clock            | KEY0        | input |
|n_reset          | KEY1        | input |
|p                | SW0         | input |
|y[1:0]           | LEDR1-LEDR0 | output |
