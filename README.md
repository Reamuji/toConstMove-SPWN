# toConstMove-SPWN

to_const alternative for move trigger usage. much more group efficient in large scale compared to the OG one

## Limitation & Issue      

- **Limited to move trigger X and Y value**
- **Delay**  
    toConstMove isn't instant. the more value you use, the longer it take to execute  
    **Range size = |a-b| in a..b**  
    it took 0.55 second for 2048 and above range size  
    0.5 for 1024 - 2047 range size  
    0.45 for 512 - 1023 range size  
    etc   
    but it's constant, which mean you can add a wait() in your program if you want to move something alses at the same time
- **Range limited to 4095**  
    will be fixed once while loop is implemented  
    you can increase it is by adding a value in the file
#### Issue in specific case      
- **Posible issue when range low value is a number of power 2 (2,4,8,16,etc)**  
    might problematic :   
    **8**..90  
    68..**32**  
- **Small jitter when the low value is negative**  
    if the move duration is 0 the obj will teleport to somewhere else for 1 frame before going to destination  
    else, the obj will jitter while moving (noticable when the obj supposed to not moving)  
- **You have to seperate the move() in some cases**  
    if the move() x or y parameter is not 0 or a 'toConstMove' return, it might need to be seperated.  
    ```spwn
    counter = count.toConstMove(..100)
    1g.move(counter + 10,5,1)
    ```
    is not allowed. because both the 'counter + 10' and '5'. but '1' is okay, because it's for the duration parameter.  
    i will explain it further below

>if you only need to implement it in a small range. to_const might be better for you.  

## Syntax

This function is designed to be as similiar as posible to the to_const function. however there are some difference.   
There are 2 rules in this function usage :
1. you have to put it inside it's own function.  
2. you have to add '->' when executing the function.  

### Syntax :

```spwn
functionName =(){
    number = counter.toConstMove(range)
    group.move(/*insert move function parameter*/)
}

//code
-> functionName()
//more code
```

if you only call it once in your program you could just write it as
```spwn
//code
-> (){
    number = counter.toConstMove(range)
    group.move(/*insert move function parameter*/)
}()
//more code
```
### Example :

```spwn
count = counter(1i)
wait(0.5)

tcm =(){
    cntConst = count.toConstMove(..100)
    1g.move(cntConst,0,0)
}

-> tcm()
2g.move(0,10,0)
wait(0.5)
-> tcm()
wait(0.5)
-> tcm()
```

if you only call it once
```spwn
count = counter(1i)
wait(0.5)

2g.move(0,10,0)
wait(0.5)
-> (){
    cntConst = count.toConstMove(..100)
    1g.move(cntConst,0,0)
}()
wait(0.5)
2g.move(0,10,0)
```

>__you can find more inside the example file__  
and if you have difficulty implementing this in your code, you can contact me on discord Reamuji#9847 or tag me in the SPWN Server. but im not sure ill always be there

### Move() seperation rules

I mentioned earlier that if the move() x or y parameter is not 0 or a 'toConstMove' return, it might need to be seperated. actually there's one other input that not require seperation, which is 'toConstMove * /<number/>'. both multiplication and division works here.  
for example:
```spwn
counter = count.toConstMove(..100)
1g.move(counter *10,0,2.5)
```
This don't need to be seperated

now, i will take the example on limitation and issue and seperate it
 ```spwn
counter = count.toConstMove(..100)
1g.move(counter + 10,5,1)
```
first i'll let's write it with proper syntax (we're not seperating it yet)
 ```spwn
//code
-> (){
    counter = count.toConstMove(..100)
    1g.move(counter + 10, 5, 1)
}()
//more code
```
finally, time to seperate it
```spwn
//code
-> (){
    counter = count.toConstMove(..100)
    1g.move(counter, 0, 1)
}()
wait(0,35) //to compromise the toConstMove delay, so the start move together
1g.move(10,5,1)
//more code
```
it's that simple
    
if you want to call it multiple time 
```spwn

move1g = (){
    -> (){
    counter = count.toConstMove(..100)
    1g.move(counter, 0, 1)
    }()
    wait(0,35) //to compromise the toConstMove delay, so the start move together
    1g.move(10,5,1)
}

//code
-> move1g()
wait(2)
-> move1g()
//more code
```
    
## Brief technical explanation

### How does it works ?

Thing you might not know is that, this is essentially a binary converter  
This utilize the fact that any number could be broken up into sum of a bunch 2^i, without ever need repating a number in 'i'

| number | 2^i sum    | i       | binary  |
|--------|------------|---------|---------|
| 1      | 1          | 0       | 0000001 |
| 2      | 2          | 1       | 0000010 |
| 3      | 1 + 2      | 0, 1    | 0000011 |
| 4      | 4          | 2       | 0000100 |
| 9      | 1 + 8      | 0, 3    | 0001001 |
| 69     | 1 + 4 + 64 | 0, 2, 6 | 1000101 |

so as long a number exist in binary, it will be posible to broken up. (which mean all number is posible)

what this function does is like converting a number to binary, and then convert it back to decimal.  
but when converting back to decimal, we were activating a bunch of move trigger instead of adding number.  
we know that sum member would be 2^i, so we could generate move trigger wih value 1,2,4,8,16 etc.

the actual code skip a lot of step for efficiency, but that basically how it's work

### Why i can't use it on other trigger ?

**Short answer :**  
trigger behavior  

**Long answer :**  
when activating a bunch of trigger with same type and same group target, only move trigger would stack the value.  
other trigger would just execute one trigger,maybe randomly chosen, idk. you need to assign multiple group to the target obj to do it.  
even though it's still more group efficient than to_const, the function would be complicated to make and use
