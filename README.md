# toConstMove-SPWN

to_const alternative for move trigger usage. much more group efficient in large scale compared to the OG one  
>Recomended for range with size 32 and above  
if you only need to implement it in a small range, to_const is better.  

Idk about other os, but in windows, you can just copy the ToConstMove folder and paste it in inside the libraries folder which located at ```C:\Program Files\spwn\libraries```.  then you could import it with only adding ```import ToConstMove``` in your code  

## Limitation     

- **Limited to move trigger X and Y value**
- **Small jitter when the low value in range is negative and the move duration isn't 0**  
    in that case, the obj will jitter while moving. but will only noticable when the move value is low or 0  
    it's actually consequence of how move in GD work. it can't be fixed while keeping current method  
- **The other coordinate value have to be 0**
    ```spwn
    1g.move(count.toConstMove(0..100), 10, 1)
    ```  
    in ths case, the ```X = count.toConstMove(0..100)``` and ```Y = 10```.  
    this isn't allowed. the Y have to be 0. if you want, you can do this instead 
    ```rs
    -> 1g.move(count.toConstMove(..100),0,1)
        // we add '->' so both move activated together
    1g.move(0,10,1)
    ```  
- **You can't do non distributive transformation to the value**  
    you don't have to worry if your object will move exactly how much the number stored inside the counter
    
    just in case you forget, distributive is when  
    ```a*(b+c) = a*b + a*c```  
    in this case, multiplication is distributive.
    ill explain this further down below


## Standard syntax :

This function is designed to be as similiar as posible to the to_const macro. but there's 1 rule : **you should write it inside the move macro itself**
#### Syntax :
```rs
//code
    group.move(counter.toConstMove(range),0,<other move macro parameter>)
//more code
```
0 is the Y value. you can flip the X and Y if you want.  
the "other move macro parameter" is optional. just like regular move syntax

if you call it multiple time, it recomended to write it this way for group efficientcy
```rs
macroName =(){
    group.move(counter.toConstMove(range),0,<other move macro parameter>)
}

//code
macroName()
//more code
macroName()
//even more code
```

### Example :

one time call
```rs
count = counter(1i)

2g.move(0,10,1)
1g.move(count.toConstMove(..100),0,2,EASE_IN)
2g.move(0,10,1)
```

multiple time call
```rs
count = counter(1i)

macro =(){
    1g.move(0,count.toConstMove(..100))
}

macro()
2g.move(0,10,0)
wait(0.5)
macro()
wait(0.5)
macro()
```

>if you have a question or difficulty on implementing this in your code, you can contact me on discord Reamuji#9847 or tag me in the SPWN Server.

### Distribution rules

let's take the equation i mentioned earlier   
```a*(b+c) = a*b + a*c```  
so we know form this that multiplication is Distributive over addition. which mean this code is valid :
```rs
1g.move(5 * count.toConstMove(..100),0)
```
now lets take a non distributive transformation  
```a+(b+c) ≠ a+b + a+c```  
addition isn't distributive over addition, which mean this code is not allowed : 
```rs
1g.move(3 + count.toConstMove(..100),0,)
```
you can use this method to check if something is "distributive over addition" or not  
```sin(b+c) ≠ sin(b) + sin(c)``` sin is not distributive   
```2^(b+c) ≠ 2^b + 2^c``` exponentiation is not distributive   
```10*(b+c)/4 = 10*b/4 + 10*c/4``` multiplication then division is distributive   
    
## Brief technical explanation

### How does it works ?

Unlike toConst that return the value directly, this macro is actually just returning a bunch of value that add up to the original value.  
how are the value broken up ? using a binary converter.  
with binary, we can broken up the value into a bunch of ```2^i``` where ```i``` is any real number. and ```i``` will never be repeated

| number | 2^i sum    | i       | binary  |
|--------|------------|---------|---------|
| 1      | 1          | 0       | 0000001 |
| 2      | 2          | 1       | 0000010 |
| 3      | 1 + 2      | 0, 1    | 0000011 |
| 4      | 4          | 2       | 0000100 |
| 9      | 1 + 8      | 0, 3    | 0001001 |
| 69     | 1 + 4 + 64 | 0, 2, 6 | 1000101 |

Notice that ```i``` is correspond to which digit is equal to 1 in binary form  
what this function does is like converting a counter to binary, and then convert it back to decimal.  
but when converting back to decimal, we also returning ```2^i``` value while were adding it back to the counter. 

the actual code skip a lot of step, but that basically how it's work

### Why i can't use it on other trigger ?

**Short answer :**  
trigger behavior  

**Long answer :**  
when activating a bunch of trigger with same type and same group target, only move trigger would stack the value.  
other trigger would just execute one trigger,maybe randomly chosen, idk. you need to assign multiple group to the target obj to do it.  
even though it's still more group efficient than to_const, the function would be complicated to make and use
