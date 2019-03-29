# Hangman-Player
This is a simple hangman-like solver. It guesses the word you have given it, as long as it is one of the 10,000 most common words as determined by [worldlywisdom's list](https://github.com/first20hours/google-10000-english) (which itself is determined by Google's Trillion Word Corpus).

# Methodology:
When you send the program a word, it takes in its length and sorts through the 10,000-word DataFrame to determine all the words that match the length of your input word. 

Using this new subset of words, it goes through each and adds up the frequency of each letter found to a dictionary of all the letters. When completed, it shows the frequency distribution of each letter that could be in your word.

The letter with the greatest frequency is determined to be the primary guess, and is pseudo-guessed. I say pseudo-guessed because there's no user input, as should be in a real hangman-like game; instead, I have a method that checks if the guessed letter is in the word. If the letter is in the word, the method returns the positions of that letter in the word; if the letter isn't, the letter with the next highest frequency is guessed. This continues until a correct guess is made.

Here's my favorite part, and one of the most challenging aspects of the code for me: When a letter is guessed correctly, we have a subset of the solved word. (For example, if your word is 'school' and the program guessed 'o', now it knows that the word is something like '_ _ _ o o _'.) Here, we have additional information to narrow down the possible words that yours could be. However, determining a way to use this information to narrow down the list of possible words was a challenge, as I had no idea how to even approach the problem. I tried various approaches, such as storing the subset as a list, but then indexing became an issue, as I had a letter but no place for it. Eventually, I realized I could use a dictionary, storing the index as the key and the letter at the value, making for a simple iterative check on words by cycling through the dictionary and only checking at the indices that had known values. This seemed to be the fastest and simplest approach to sorting through words.

This worked pretty well, and I was happy with the results I was seeing from the program. It was able to guess words within a fraction of a second, and I averaged ~50% success (meaning every other guess was right). I thought that was a pretty decent outcome, but I wondered if I could do better. After all, I do have the words in order of their frequency in the English language, so words at the top of the list should be more common than words at the bottom. With this in mind, I tried something simple-ish.

I began assiging weights to words; I gave words at the top of the list 2x the weight as words at the bottom, and tried running a few words to see if it took less time. To my surprise, it did! I was excited, so I considered running a more scientific test. I measured the success as my GSR(Guesses to Successes Ratio), meaning I'd divide the number of guesses I made by the number of unique letters in the word to find how accurate my program was. I had two weights to optimize, which I called "Slope" and "Intercept". These names came because I calculated the weights as (intercept * number_of_words) + (slope * position_in_list), which is another way of writing y=mx+b, where y is GSR, m is 'slope', and b is 'intercept*number_of_words'. 

I'd randomly sample 500 words from the 10,000-word list, run them through the program, and see what my success rate was. I'd run that test 5 times, to get a more accurate understanding and reduce variation. Then, I'd change the weights and try to determine their optimal values to make the program the most accurate guesser it could be. I knew that if I had just one weight to optimize, the best way would be to take two guesses, see which is better, and move in that direction, until I was reasonably close to an optimal solution. 

However, I had two weights, so I needed to take a different approach. At first, I considered trying to optimize one value to a specified benchmark, then optimizing the other, the optimizing the first to a smaller benchmark, and continuing until the program was sufficiently accurate. I tried to test this out on a fake set with a fake program. The program would also take in two weights and would output the absolute value of the distance from two arbitrary values, like 6.2 and 7.5.  Then, I used my method, starting from 0 and 0, to try and find the optimal values of the weights. However, it took an unreasonably large number of calculations, and I realized something that bummed me out for a while. Each actual test, finding a single GSR for a slope-intercept pair would take an hour. An hour! I couldn't do 200 tests to find the optimal values, because school starts in a few days and I can't just let it keep testing in the background for two weeks, just to know if my model was right or accurate enough. However, I realized that this just meant I would be pushed to find as accurate and efficient a model as I could, or I wouldn't be able to get the answer I wanted to find after all this work. So, I tried a new way of finding the optimal value. I've copied below what I wrote out in my code when trying to understand what I was doing.


```
"""          
                 a   b
                     d   e
                 c       g
                     f          
                                       
                                               
                                                    •
                        
                        
first, we take the three points as our starters
we find the rate of change between them, (∆GSR/∆slope or ∆intercept)
move in the direction of the change between points times the rates of change
a(0,0) = 10
b(1,0) = 9
c(0,1) = 9.5
∆ab = 1/1   = 1               #a - b
∆ac = 0.5/1 = -0.5           #a - c
(b-a)*∆ab = <1,0>*1   = <1,0>
(c-a)*∆ac = <0,1>*0.5 = <0,0.5>
d(1,0.5) = 8.5
e(2,0.5) = 7.5
f(1,1.5) = 8
∆de = 1/1   = 1
∆df = 0.5/1 = 0.5
(e-d)*∆de = <1,0>*1   = <1,0>
(f-d)*∆df = <0,1>*0.5 = <0,0.5>
<1,0> + <0,0.5> = <1,0.5>
<1,0.5>+<1,0.5> = <2,1>
g(2,1)
...
...
...



"""
```

A weird way of approaching the problem, but I figured this would be a reasonable way to try and get the answer as quickly as possible. I tried testing this model out on my sample program, and I got decent results, until I got close to the optimal value. Because I was incrementing by 1, I would often be .5 away from the right answer and just be bouncing back and forth between -0.5 and +0.5. Ok. I need to reduce the increment as I get closer to the right answer, and I could probably increase my increment when I'm starting. I tried to use the square root of the output as my change factor; it got smaller as I approached the optimum, and was sufficiently large at the start to reduce the number of calculations. I tried various "versions" of this function, such as 2*sqrt, sqrt(x-1), and even of the log function, but the most efficient was the basic square root function. With this in mind, I decided I had to start testing. I was on day 3 of this, and I'd devoted so many hours to understanding this; if I didn't commit now, I'd just keep pushing myself off and trying new and different methods without understanding how effective they actually were. So, at that point, I said screw it, and tried my running my optimization method on the real hangman player. 

It's been 3 days since I started running it. Because I keep accidentally closing my laptop, calculations have taken much longer than they should. (My laptop doesn't run tests when it's closed.) However, I've realized something pretty embarrassing. The calculations I'm running are absolutely useless. I'm depending on my samples of the dataset, which are equally distributed through the set. This means I'm not actually finding the perfect weights for real-life scenarios where people are giving words and the program is guessing them; instead, the situation I'm "optimizing" is picking a random word from the list and trying to guess it. These situations are quite different, and the one I'm solving for isn't the one I should be.

If I actually wanted to optimize the weights, I'd need real data. Usage from people who are actually playing hangman and giving random words that I can use as true testing data. Therefore, my next task is to try and use the fundamental code here to build an android app that I'll publish onto the Play Store. That app will send me usage statistics and the words that real people use when playing hangman so I can collect real data that can be used to find real values for the weights. 

I hope to learn how to access a program in python from an app in Java/Android Studio, to learn how to send and receive data from an application, and to learn a better way for finding the optimal solution of two weights. Let's see how this goes!

# 
# 
