# THE DIAMOND PROBLEM SOLVING IN APL

The diamond problem goal is to draw a character-based shape of a diamond, with A at the top, like this:

```
  A
 B B
C   C
 B B
  A
```
This problem was a pretext in a training about TDD, where some simple code had to be elaborated in Java, and various methods were discussed. This one is genuine. Somehow, it takes its inspiration from functional programming. APL is naturally handling vectors, so its style of programming is more on the declarative and functional side, than on the imperative side. Well, you will make your own opinion soon. Also consult Dijkstra's judgement on APL.

"APL is a mistake, carried through to perfection. It is the language of the future for the programming techniques of the past: it creates a new generation of coding bums."

I, proudly, am one of those...

Powerful operators exist to transpose, shift, mirror matrices. And as the shape, i.e. the number  of  dimensions  and  their  value  is critical, operators exist to read the shape, or to reshape data. 

The approach to the diamond problem is to recognise the two symmetries of the shape, and focus on one quadrant. This is the first "difficult" step of the design, the choice of a strategy. Let's use the diamond up to 'C' as an example. 'C' will also be called here the 'widest row character' .

```
A
 B 
  C
```

The  approach  in  APL  will  try  to generate a numerical representation of the 
problem based on a mapping such as:

* 0 for ' '
* 1 for 'A'
* 2 for 'B'
* 3 for 'C'

Therefore, the target is to generate this matrix:
```
1 0 0
0 2 0
0 0 3
```

The value 3 is easily obtained as the 1-based index of 'C' in the alphabet:

```apl
    	N←(A←'ABCDEFGHIJKLMNOPQRSTUVXYZ')⍳'C' 
		N
3
```
The practice of APL helps in decomposing the target matrix, but admittedly, this 
step is probably the least obvious  one in the design.  The target matrix can be 
obtained  with  a  cell-to-cell  product  of two easy  matrices. This is not the 
mathematical matrix product noted differently in APL. 
```
1 0 0   1 2 3
0 1 0 x 1 2 3
0 0 1   1 2 3
```

The  first  matrix  is the identity matrix, and can be generated using an idiom, 
based on the pattern "1 followed by the same number of 0 as the width", 1 0 0 0 
in the case of 3. The pattern is reused as many times or truncated as needed. 

```apl
       3 3 ⍴ 1 0 0 0 
1 0 0
0 1 0
0 0 1
```

With the variable N earlier defined as 3, this is noted:

```apl
		(N,N)⍴1,N⍴0
```

A shorter way involves a cartesian  product,  comparing  a  vector of 3 distinct 
element to itself with the outer product operator:
```apl
		∘.=
```

For example:
```apl
		V∘.=V←⍳3 
```
is another way (with some extra calories: it's richer than it needs to be) to produce the identity matrix. It may translate to: Only the diagonal has row = column

Same for the second matrix, but it is based on the pattern "all the integers up 
to n", 1 2 3 in the case of 3. Fortunately, APL has an "index generator":

```apl
       	⍳3
1 2 3
       	3 3 ⍴ ⍳3
1 2 3 
1 2 3
1 2 3 
```

With the earlier defined variable N this is noted:
```apl
		(N,N)⍴⍳N
```

Let's call M our model, so we have established:

```apl
		M←((N,N)⍴1,N⍴0)×(N,N)⍴⍳N
``` 
At this point we have a quarter of the model, and to build the full model of the
diamond,  we must  apply  two  symmetries,  with the vertical mirror '⌽'' and the horizontal mirror '⊖' operators, and some 
trimming of duplicate columns/lines, achieved with the Drop operator '↓', and paste the two parts together:

```
0 0 1|1|0 0
0 2 0|0|2 0 
3 0 0|0|0 3
      ^ 
      to be removed
```
Let's call T like top, the result

T is the vertical mirror of M, pasted with a copy of M with 0 line and the first column dropped.

```apl
		T←(⌽M),(0 1)↓M←((N,N) ⍴ 1,N ⍴ 0)×(N,N)⍴⍳N
```

Next step, we want T pasted on top of its horizontal mirror, less the top line
* The dash with a cedilla, is the 'paste on top of' operation
* One line, zero column dropped from horizontal mirror of T. T is unchanged, a copy of it is modified.
```
0 0 1 0 0
0 2 0 2 0
3 0 0 0 3
---------
3 0 0 0 3 < to be removed
---------
0 2 0 2 0
0 0 1 0 0
```
Let's call D like diamond the result:

```apl
	D←T⍪(1 0)↓⊖T←(⌽M),(0 1)↓M←((N,N) ⍴ 1,N ⍴ 0)×(N,N)⍴⍳N←(A←'ABCDEFGHIJKLMNOPQRSTUVWXYZ')⍳'C'
    	D
0 0 1 0 0
0 2 0 2 0
3 0 0 0 3
0 2 0 2 0
0 0 1 0 0
```

Next step, an ad-hoc vector (or  string) of characters is composed with a space followed by the  subset of the alphabet up to the "widest row" character giving ' ABC', whith the indices starting at 1:

* 1 for ' '
* 2 for 'A'
* 3 for 'B'
* 4 for 'C'

Look at first line of code when we defined N, A is also defined as the whole alphabet.
```apl
	' ',N⍴A
```

In APL, a vector can be reshaped by indexing, the result will take the shape of the indexing matrix.
As D is zero-based, we'll add 1 to it and index that ad-hoc vector:

```apl
	(' ',N⍴A)[1+D←T⍪(1 0)↓⊖T←(⌽M),(0 1)↓M←((N,N) ⍴ 1,N ⍴ 0)×(N,N)⍴⍳N←(A←'ABCDEFGHIJKLMNOPQRSTUVWXYZ')⍳'C']
```

## Closing note

* Scroll through to the right end of the above line of code to select and copy it. It's 103 characters long.
* Go to http://tryapl.org and paste it, followed by a "carriage return" to see the result
* Despite the cryptic notation, APL code can very easily be read as an English phrase. Because of its unusual precedence order, from right to left, it is easy to build code by extending it progressively from the left end:
 * In APL, 2 x 1 + 1 is 4

* APL on the desktop
	- GNU APL (Dr. Jürgen Sauermann) has been used to test this code.
	- DyalogAPL is an advanced implementation of APL with a free community version for Windows. 
   		* TryAPL.org is their portal.
   		* It supports the alpha-omega notation, which allows the definition of lambdas: 
```apl
 		{⍵×⍵}5
25
  ```
 - There is a Javascript based interpreter ngn-apl, which assumes 0 as the base of indexes: ⎕IO is zero and can't be reassigned without a DOMAIN ERROR. Go to https://repl.it/languages/APL to try the adjusted code:
```
	A[T⍪(1,0)↓⊖T←(⌽M),(0,1)↓M←((N,N)⍴(1,N⍴0))×1+(N,N)⍴⍳N←(A←' ABCDEFGHIJKLMNOPQRSTUVWXYZ')⍳'C']
```
 - On  macOS, Ukelele is a keyboard configurator tool.

