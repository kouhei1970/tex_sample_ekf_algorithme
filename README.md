# 拡張カルマンフィルタ

拡張カルマンフィルタののあらましをご紹介します。

拡張カルマンフィルタは非線形な状態方程式と観測方程式を、動作点で線形化しつつカルマンフィルタを使っていくです。

拡張カルマンフィルタにおいて推定されるkステップ目の推定量は以下です。


- 状態の推定値 $\hat{\boldsymbol{x}}\_k$ 

- 誤差共分散の推定値 $\hat{\boldsymbol{P}}\_k$ 

## 計算にあたり必要とするもの

- state $\hat{\boldsymbol{x}}\_k$

- 状態方程式 $\dot{\boldsymbol{x}}={\boldsymbol{f(x)}}+{\boldsymbol{G}}{\boldsymbol{w}}$

- 観測方程式  ${\boldsymbol{z}}={\boldsymbol{h(x)}}+{\boldsymbol{v}} $

- システムノイズの共分散行列 ${\boldsymbol{Q}}$

- 観測ノイズの共分散行列 ${\boldsymbol{R}}$

- 状態と誤差共分散の初期値 ${\boldsymbol{x}\_0}$、${\boldsymbol{P}\_0}$


## したごしらえ

### 状態方程式の離散化

カルマンフィルタも拡張カルマンフィルタも離散系のとして用います。
したがって、微分方程式（連続時間）で定義されている状態方程式を離散化する必要があります。
最も簡単な方法としては、微分を短い時間${\Delta t}$で近似したものを用いて、離散化することです。

まず、状態方程式において、微分を数値微分と置き換えると


$$
\begin{eqnarray}
\frac{ \boldsymbol{x}_{k+1} - \boldsymbol{x}_{k} }{\Delta t} = {\boldsymbol{f}({\boldsymbol{x}_k})}  +{\boldsymbol{G}}{\boldsymbol{w}}
\end{eqnarray}
$$



${\boldsymbol{x}}_{k+1}$について整理すると

<div align="center">$
\begin{eqnarray}
{\boldsymbol{x}}_{k+1}  = {\boldsymbol{x}}_{k}+{\boldsymbol{f}({\boldsymbol{x}_k})} {\Delta t}  +{\boldsymbol{G}}{\boldsymbol{w}}{\Delta t}
\\
\\
\end{eqnarray}
$</div>

ここで、次の様な置き換えを行うと（添字$d$はDiscrete（離散）の頭文字dです。）

<div align="center">$
\begin{eqnarray}
{\boldsymbol{f}_d \boldsymbol{(x}_k  \boldsymbol{)}}={\boldsymbol{x}}_{k} + {\boldsymbol{f(x}_k \boldsymbol{)}} \Delta t
\\
\\
\end{eqnarray}
$</div>

<div align="center">$
\begin{eqnarray}
{\boldsymbol{G}}_k={\boldsymbol{G}} \Delta t
\\
\\
\end{eqnarray}
$</div>

離散化状態方程式は次の様に、見た目は離散化前と同じ様な式になります。

<div align="center">$
\begin{eqnarray}
{\boldsymbol{x}}_{k+1}={\boldsymbol{f}_d \boldsymbol{(x}_k \boldsymbol{)} }+{\boldsymbol{G}}_k {\boldsymbol{w}}
\\
\\
\end{eqnarray}
$</div>


##### ヤコビアンの導出

ベクトル関数をベクトルで偏微分したものをヤコビアンと呼びます。
ヤコビアンは行列になります。
１次元で言えば接線の傾きに当たるものがヤコビアンです。

曲線のある場所での接線とその方程式を求める作業が線形化ですので、状態方程式と観測方程式のヤコビアンを求めることが
線形化作業そのものになります。

状態方程式の${\boldsymbol{f}\_d}({\boldsymbol{x}\_k})$と観測方程式の${\boldsymbol{h}}({\boldsymbol{x}}\_k)$を状態量ベクトルで編微分して
以下の様に、それぞれを${\boldsymbol{F}}({\boldsymbol{x}}\_k)$、${\boldsymbol{H}}({\boldsymbol{x}}\_k)$ とします。

<div align="center">$
\begin{eqnarray}
{\boldsymbol{F}}({\boldsymbol{x}}_k) =\frac{\partial \boldsymbol{f}_d}{\partial \boldsymbol{x}_k} 
={\boldsymbol{I}}+\frac{\partial \boldsymbol{f}}{\partial \boldsymbol{x}_k} \Delta t

\\
\\
\end{eqnarray}
$</div>


<div align="center">$
\begin{eqnarray}
{\boldsymbol{H}}({\boldsymbol{x}}_k) =\frac{\partial \boldsymbol{h}}{\partial \boldsymbol{x}_k} 
\\
\\
\end{eqnarray}
$</div>

特に${\boldsymbol{F}}({\boldsymbol{x}}\_k)$については、離散化された状態方程式のヤコビアンなので注意してください。

##### ヤコビアンについて

ベクトルをベクトルで編微分すると行列になり、ヤコビアンと呼ばれています。具体例は次の様になります。

微分される関数ベクトルを次の様なものとします。

<div align="center">$
\begin{eqnarray}
{\boldsymbol{f}} =
\begin{bmatrix}
f_1 &f_2 & f_3
\end{bmatrix}^\mathsf{T}
\\
\\
\end{eqnarray}
$</div>

微分するベクトルを次な様なものとします。

<div align="center">$
\begin{eqnarray}
{\boldsymbol{x}} =
\begin{bmatrix}
x_1 &x_2 & x_3 & x_4
\end{bmatrix}^\mathsf{T}
\\
\\
\end{eqnarray}
$</div>

すると、ヤコビアンは次の様な行列になります。

<div align="center">$
\begin{eqnarray}
\frac{\partial \boldsymbol{f}}{\partial \boldsymbol{x}} =
\begin{bmatrix}
\frac{\partial f_1}{\partial x_1} & \frac{\partial f_1}{\partial x_2} &\frac{\partial f_1}{\partial x_3} &\frac{\partial f_1}{\partial x_4}\\ 
\frac{\partial f_2}{\partial x_1} & \frac{\partial f_2}{\partial x_2} &\frac{\partial f_2}{\partial x_3} &\frac{\partial f_2}{\partial x_4}\\
\frac{\partial f_3}{\partial x_1} & \frac{\partial f_3}{\partial x_2} &\frac{\partial f_3}{\partial x_3} &\frac{\partial f_3}{\partial x_4}\\
\end{bmatrix}
\\
\\
\end{eqnarray}
$</div>



####

拡張カルマンフィルタは以下に示す、観測更新と予測更新を各ステップごとに繰り返していきます。

流れを大雑把に書くと次の様になります。


1. カルマンゲインを計算
2. 観測値とカルマンゲインによって推定値を修正して推定値を得る 
3. 前のステップで得られた状態量と誤差共分散により、予測値を得る

一つのステップの中で、推定と予測の二つの部分に分られます。

ここでは、推定された値にはハット（＾）を、予測された値にはバー（ー）を記号の上につけることにします。


##### 観測更新

###### 観測方程式ヤコビアンの準備

<div align="center">$
\begin{eqnarray}
{\boldsymbol{H}}_k =\left. \frac{\partial \boldsymbol{h}}{\partial \boldsymbol{x}_k} \right|_{{\boldsymbol{x}}_k = \bar{\boldsymbol{x}}_k} 
\\
\\
\end{eqnarray}
$</div>

###### カルマンゲイン更新
<div align="center">$
\begin{eqnarray}
{\boldsymbol{K}}_k =\bar{\boldsymbol{P}}_k {{\boldsymbol{H}}_k}^\mathsf{T} {\left\[ {{\boldsymbol{H}}_k} {\bar{\boldsymbol{P}}_k} {{\boldsymbol{H}}_k}^\mathsf{T} + {\boldsymbol{R}} \right\$}^{-1}
\\
\\
\end{eqnarray}
$</div>


###### 状態推定
<div align="center">$
\begin{eqnarray}
\hat{\boldsymbol{x}}_k =\bar{\boldsymbol{x}}_{k} + {\boldsymbol{K}}_k \left\[ {\boldsymbol{z}}_k - {\boldsymbol{h}}(\bar{\boldsymbol{x}}_k)  \right\$
\\
\\
\end{eqnarray}
$</div>

###### 誤差共分散推定
<div align="center">$
\begin{eqnarray}
\hat{\boldsymbol{P}}_k =\bar{\boldsymbol{P}}_{k} -  {\boldsymbol{K}}_k {\boldsymbol{H}}_k \bar{\boldsymbol{P}}_{k} 
\\
\\
\end{eqnarray}
$</div>


#####予測更新


###### 状態の予測
<div align="center">$
\begin{eqnarray}
\bar{\boldsymbol{x}}_{k+1} =\boldsymbol{f}_d ( \hat{\boldsymbol{x}}_{k} ) 
\\
\\
\end{eqnarray}
$</div>

###### 状態方程式ヤコビアンの準備

<div align="center">$
\begin{eqnarray}
{\boldsymbol{F}_k} =\left. \frac{\partial \boldsymbol{f}_d}{\partial \boldsymbol{x}_k} \right|_{{\boldsymbol{x}}_k = \bar{\boldsymbol{x}}_k} 
={\boldsymbol{I}}+\left. \frac{\partial \boldsymbol{f}}{\partial \boldsymbol{x}_k} \right|_{{\boldsymbol{x}}_k = \bar{\boldsymbol{x}}_k}  \Delta t
\\
\\
\end{eqnarray}
$</div>


###### 共分散の予測
<div align="center">$
\begin{eqnarray}
\bar{\boldsymbol{P}}_{k+1} ={\boldsymbol{F}_k} \hat{\boldsymbol{P}}_{k} {{\boldsymbol{F}}_k}^\mathsf{T}+ {\boldsymbol{G}_k}{\boldsymbol{Q}}{\boldsymbol{G}_k}^\mathsf{T}
\\
\\
\end{eqnarray}
$</div>

