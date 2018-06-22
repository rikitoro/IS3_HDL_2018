IS3 マイクロコンピュータ基礎 HDL実習

# 3章 レジスタの設計

本章では、SystemVerilogでレジスタを設計する方法を学びます。

---
## レジスタ

図3.1に示したクロック同期の4ビットレジスタは、リスト3.1の register モジュールのように記述できます。

![4ビットレジスタ](./assets/register.png "4ビットレジスタ")

<図3.1 4ビットレジスタ>


<リスト3.1 register モジュール (4ビットレジスタ)>

```SystemVerilog
module register(
  input         clock,
  input   [3:0] d,
  output  [3:0] q
);

  always_ff @ (posedge clock) begin // (1) clockの立ち上がりのタイミングで起動
    q <= d; // (2) qにdの値を代入する(ノンブロッキング代入)
  end

endmodule
```

レジスタのようにフリップフロップによって構成されるような回路は always_ff 文を使って設計することができます。
always_ff 文は @ 以下に示された信号が変化したときにbegin と end で囲われた部分が実行されるような回路を構成します。
リスト3.1の(1)の部分では、clock 信号の立ち上がり(posedge)のタイミングが指定してあります。
すなわち、clock 信号の立ち上がりのタイミングで(2)の`q <= d`が実行されます。
これは信号 q に信号 d の値を代入することを示しています。

この register モジュールの動作をまとめると、clock 信号の立ち上がりのタイミングで入力信号 d の値を取り込み、その値を q に保持し出力することになります。
図3.2にこのregister モジュールの動作例をタイムチャートで示します。
上記の動作を確認してください。


![register モジュールの動作例](./assets/timechart_register.png)

<図3.2 register モジュールの動作例>

なお、今回リスト4.1の always_ff 文で用いた `q <= d` のような `<=` による代入はノンブロッキング代入と呼ばれます。
フリップフロップによって構成されるような回路を設計するときは原則としてノンブロッキング代入を用いるのが良い習慣です。

### 演習

リスト3.1 register モジュールを実習ボード DE0-CV に実装してその動作を確認しましょう。

register モジュールの入出力信号は表3.1のように DE0-CV の入出力デバイスに割り当てましょう。

<表3.1 register モジュールの入出力のデバイスへの割り当て>

|信号名|割り当てデバイス|入出力|
|------|----------------|------|
|clock | KEY0           | input |
|d[3:0]| SW3-0          | input |
|q[3:0]| LEDR3-0       | output |

---
## 非同期リセット付きレジスタ

![非同期式リセット付き4ビットレジスタ](./assets/register_ar.png "非同期式リセット付き4ビットレジスタ")

<図3.3 非同期式リセット付き4ビットレジスタ>

```SystemVerilog
module register_ar( // asynchronous reset
  input         clock,
  input         n_reset, // active low (0になったらリセット)
  input   [3:0] d,
  output  [3:0] q
);

  always_ff @ (posedge clock, negedge n_reset) begin
    if (n_reset == 1'b0) begin
      q <= 4'b0000;
    end else begin
      q <= d;
    end
  end

endmodule
```
