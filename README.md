# Dark automated match engine.
Non-custodial order book over darkfi blokchcain.
In this model the traders doesn't have to interact after the swap for transaction signature, instead settlement node which owns the coins, can transfer the swapped tokens directly, since the first part of the swap is already done in the exchange transaction by burning the base coins.

## order type
the match order only accept limit orders.

## exchange transaction
Assume a trader owns a coin c of Token $t_1$ in order to add a an order in the order book with public key $pk^{matcher}$, the trader burns c, mint new $c^\prime=(pk^{matcher}_1,v_1,t_1, spend hook_1, user data_1, coin blind_1, token blind_1)$, create an order with quote token id $t_2$ commit to it with $token blind_2$, quote rate p (quote price per base price), timeout duration, $\tau$, constrain order commit for order o = ($t_2$, p, $\tau$), the proof need to commit to withdraw public key $pk^{withdraw}$ that can be used in case of withdraw or settlement.
note, no need to reveal $pk^{matcher}$ in the public inputs to restrict the swap to specific exchange since the coin is minted with that matcher pk

### send the gas fee in the order details (TODO)
exchange proof need to commit to gas fee value, gas fee token.

## match proof
It's a proof of knowledge that the match engines have access two coins with non-negative spread (this is two steps, the coin values are sufficient, and the rate matches), two coins are of the opposite direction, the counter party token is the desired token.
for a match between two parties left (l), and right (r) prove konwledge to two coins l, r, such that:

base token of the left token is the same as quote of the right token, quote of the left token is the same as base of the right token, the exchange rate of both parties has non-negative spread.

- $commit(t^{l}_1, tokenblind^{l}_1) == commit(t^{r}_2, tokenblind^{r}_2)$
- $commit(t^{l}_2, tokenblind^{l}_2) == commit(t^{r}_1, tokenblind^{r}_1)$

the following three rules make sure that the two parties has non-negative spread.

- $p_1p_2>=1$
- $v^{r}_2 >= v^{l}_1$
- $v^{l}_2 >= v^{r}_1$

constrain, and reveal the two coins used.

### spread difference (TODO)
the spread should be added to the book, commit to the spread difference value, it will be spent along with the fee transaction upon swap success.

## order settlement transaction

settlement node, would combine match proof with transfer proofs in a single transaction.

## withdraw transaction

trader who issued a exchange transaction can withdraw, and spend the spend hook minted token as long as the settlement tx isn't broadcasted yet, or if the order timeout duration is passed.

### exchange transaction spend hook

if the duration $\tau$ specified in the order is timeout, the spend hook is executed, and the a coin of the same value is minted.

## cancel order

at any moment as long as the order isn't settled, the trader can cancel the order by minting a new coin, or if the order duration is timedout, spend hook contract can mint new Coin of the same Token and value back to the Trader owned by $pk^{withdraw}$

### cancel order before timeout

the match engine need to be notified by cancel event, for this, the burn proof in the previous exchange transaction is combined with mint proof to create new transfer proof. it seems that the burn proof will be reused, and the nullifier will be issued twice, but in fact the exchange transaction will not be finalized until it's settled, with final swap transfer, and match engine fee contract executed, until then the burn proof in exchange transaction shouldn't be finalized, and it's finalized shouldn't be added to spent coins.

### on timeout

the exchange transaction is finalized, match engine ought to remove bid/ask orders, withdraw spend hook should be executed.


how can match engine remove the order if the trader cancels the order?


\begin{figure}
 \begin{tikzpicture}[
    playernode/.style={circle, draw=green!60, fill=green!5, very thick, minimum size=7mm},
    squarednode/.style={rectangle, draw=red!60, fill=red!5, very thick, minimum size=5mm},
    spendhooknode/.style={rectangle, draw=black!60, fill=red!5, very thick, minimum size=5mm}
   ]
   \node[playernode]  (1)                                     {Token A LP};
   \node[playernode]  (2)         [right=of 1, xshift=5cm]    {Token B LP};

   \node[squarednode]    (3)   [below =of 1]     {Transfer A};
   \node[squarednode]    (5)   [below =of 2]     {Transfer B};



   \node[playernode]  (7)   [below right=of 3, xshift=1.5cm, yshift=-0.5cm]    {match engine};

   \node[squarednode]  (8)  [below=of 7]    {Match proof};
   \node[squarednode]  (9)  [below=of 8]    {order settlement};

   \node[squarednode]  (4)   [below left =of 9]      {Transfer B};
   \node[squarednode]  (6)   [below right =of 9]      {Transfer A};

   \node[squarednode]  (10)     [below=of 9]    {match engine fee};

   \node[playernode]  (13) [below of=10, yshift=-1cm]    {match engine};


   \node[playernode]  (11)   [below of=4, yshift=-1cm]     {Token A LP};
   \node[playernode]  (12)   [below of=6, yshift=-1cm]    {Token B LP};

    f\draw[->] (3.south)  .. controls +(down:20mm) and +(left:50mm) .. (1.north) node [pos=1,above,font=\footnotesize] {cancel swap, or redeposite if order timedout};

    f\draw[->] (5.south)  .. controls +(down:20mm) and +(right:50mm) .. (2.north) node [pos=1,above,font=\footnotesize] {cancel swap, or redeposite if order timedout};

    f\draw[->] (9.south) -- (4.north) node [pos=1,above,font=\footnotesize] {swap transfer of Token B};
    f\draw[->] (1.south) -- (3.north);
    f\draw[->] (9.south) -- (6.north) node [pos=1,above,font=\footnotesize] {swap transfer of Token A};
    f\draw[->] (2.south) -- (5.north);
    f\draw[->] (3.south) -- (7.north);
    f\draw[->] (5.south) -- (7.north);
    f\draw[->] (7.south) -- (8.north);
    f\draw[->] (8.south) -- (9.north);
    f\draw[->] (9.south) -- (10.north);
    f\draw[->] (4.south) --  (11.north) node [pos=0.8,above,font=\footnotesize] {Swaped coin B};
    f\draw[->] (6.south) --  (12.north) node [pos=0.8,above,font=\footnotesize] {Swaped coin A};
    f\draw[->] (7.south) -- (8.north);
    f\draw[->] (10.south) -- (13.north);

 \end{tikzpicture}
 \caption{match engine contracts}
\end{figure}
