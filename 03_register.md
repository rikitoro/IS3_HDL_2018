IS3 マイクロコンピュータ基礎 HDL実習

# 3章 レジスタの設計

本章では、always_ff 文を用いてレジスタを設計する方法を学びます。

---
## レジスタ

図3.1に示したクロック同期の4ビットレジスタを設計します。
この回路はリスト3.1の register モジュールのように記述できます。

![4ビットレジスタ](./assets/register.png "4ビットレジスタ")

<図3.1 4ビットレジスタ>


<リスト3.1 register モジュール (4ビットレジスタ)>

```SystemVerilog
module register(
  input   logic         clock,
  input   logic   [3:0] d,
  output  logic   [3:0] q
);

  always_ff @ (posedge clock) begin // (1) clockの立ち上がりのタイミングで起動
    q <= d; // (2) qにdの値を代入する(ノンブロッキング代入)
  end

endmodule
```

レジスタのようにフリップフロップによって構成されるような回路は always_ff 文を使って設計することができます。
always_ff 文は @ 以下に示された信号が変化したときに、begin と end で囲われた部分が実行されるような回路を構成します。
リスト3.1の(1)の部分では、clock 信号の立ち上がり(posedge)のタイミングが指定してあります。
すなわち、clock 信号の立ち上がりのタイミングで(2)の`q <= d`が実行されます。
これは信号 q に信号 d の値を代入することを示しています。

この register モジュールの動作をまとめると、clock 信号の立ち上がりのタイミングで入力信号 d の値を取り込み、その値を q に保持し出力することになります。
図3.2にこのregister モジュールの動作例をタイムチャートで示します。
上記の動作を確認してください。


![register モジュールの動作例](./assets/timechart_register.png)

<図3.2 register モジュールの動作例>

今回、リスト4.1の always_ff 文で用いた `q <= d` のような `<=` による代入はノンブロッキング代入と呼ばれます。
フリップフロップによって構成されるような回路を設計するときは原則としてノンブロッキング代入を用いるのが良い習慣です。

### 演習

リスト3.1 register モジュールを実習ボード DE0-CV に実装してその動作を確認しましょう。

register モジュールの入出力信号は表3.1のように DE0-CV の入出力デバイスに割り当てましょう。

<表3.1 register モジュールの入出力のデバイスへの割り当て>

|信号名|割り当てデバイス|入出力|
|------|----------------|------|
|clock | KEY0           | input |
|d[3:0]| SW3-SW0          | input |
|q[3:0]| LEDR3-LEDR0       | output |

---
## 非同期リセット付きレジスタ

先ほど設計したクロック同期の4ビットレジスタに、非同期リセット機能を追加した回路の設計を考えます。
図3.3のようにリセット入力信号 n_reset を追加します。
n_reset はアクティブローとします。
つまり、通常時は n_reset には 1 が入力され、n_reset が0になったときにリセット機能が働くものとします。

この非同期リセット付き4ビットレジスタは、リスト3.2の register_ar モジュールのように記述できます。


![非同期リセット付き4ビットレジスタ](./assets/register_ar.png "非同期リセット付き4ビットレジスタ")

<図3.3 非同期リセット付き4ビットレジスタ>

<リスト3.2 register_ar モジュール(非同期リセット付き4ビットレジスタ)>

```SystemVerilog
module register_ar( // asynchronous reset
  input   logic       clock,
  input   logic       n_reset, // active low (0になったらリセット)
  input   logic [3:0] d,
  output  logic [3:0] q
);

  always_ff @ (posedge clock, negedge n_reset) begin
    if (n_reset == 1'b0) begin
      q <= 4'b0000;   // reset
    end else begin
      q <= d;
    end
  end

endmodule
```

今回の always_ff 文では起動されるタイミングとして、 clock の立ち上がりと n_reset の立下り(negedge)の両方が指定されています。
また always_ff 文中では if 文が用いられており、n_reset が0の時は q をリセット(0を代入)し、そうでない場合は d の値を q に代入することを記述しています。

このように条件により異なる動作をする回路を記述したい場合、 always_ff 文や4章で示す always_comb 文などの always 文中において、 if 文を使うことができます。
なお、always 文の外側では if 文を使うことはできません。

図3.4に register_ar モジュールの動作例をタイムチャートで示します。
n_reset の立下りのタイミングで q がリセットされていることを確認しましょう。
また、clock の立ち上がりのタイミングでも、 n_reset が0となっていれば同様にリセットが起こることに注意しましょう。

![register_ar モジュールの動作例](./assets/timechart_register_ar.png "register_ar モジュールの動作例")

<図3.4 register_ar モジュールの動作例>

### 演習

リスト3.3 register_ar モジュールを実習ボード DE0-CV に実装してその動作を確認しましょう。

register_ar モジュールの入出力信号は表3.3のように DE0-CV の入出力デバイスに割り当てましょう。

<表3.2 register_ar モジュールの入出力のデバイスへの割り当て>

|信号名|割り当てデバイス|入出力|
|------|----------------|------|
|clock | KEY0           | input |
|n_reset| KEY1          | input |
|d[3:0]| SW3-SW0          | input |
|q[3:0]| LEDR3-LEDR0       | output |


---
## 5進カウンタ(非同期リセット付き)

先ほどの非同期リセット付きレジスタの設計を応用して、
非同期リセットが付いた5進カウンタ(図3.5)を設計してみましょう。

この5進カウンタはリスト3.3の counter5 モジュールのように作成できます。

![5進カウンタ(非同期リセット付き)](./assets/counter5.png "5進カウンタ(非同期リセット付き)")

<図3.5 5進カウンタ(非同期リセット付き)>


<リスト3.3 counter5 モジュール(5進カウンタ(非同期リセット付き))>

```SystemVerilog
module counter5 (
  input   logic       clk,
  input   logic       n_reset,
  output  logic [2:0] count
);
  always_ff @ (posedge clk, negedge n_reset) begin
    if (n_reset == 1'b0) begin
      count <= 3'd0;
    end else if (count >= 3'd4) begin
      count <= 3'd0;
    end else begin
      count <= count + 1'd1;
    end
  end

endmodule // counter10

endmodule
```

動作例を図3.6に示します。
クロック信号 clk の立ち上がりが入るたびに count の値が1ずつ増えていきますが、
countの値が4の場合は次のクロックの立ち上がりで、countの値が0に戻されているのが分かります。 

![counter10 モジュールの動作例](./assets/timechart_counter5.png)

<図3.6 counter10 モジュールの動作例>
