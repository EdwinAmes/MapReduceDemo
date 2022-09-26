# Introduction #

I based this demo on an example from the [original Google paper](https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf) that introduced the map reduce divide-and-conquer style approach or algorithm to widespread use. The demo counts the number of times each word appears in a given text.

The first section, "Concepts", is intended for a general audience. It covers map-reduce ideas and motivation. Fellow professionals will recognize some simplifying assumptions.

The "Technical Details" section explains the simplifying assumptions and lays out the Map-Reduce Demo app design.

# Concepts #

For decades computing power doubled about once every two years. To solve problems faster you got a faster computer. Back in 1965, Gordon Moore, an Intel executive, observed that the number of components doubled roughly every year. In 1975 he revised his estimate to once every two years. "Moore's law" described and defined the relentless advance driving the computer revolution, upending life as we knew it. For years the industry coordinated its efforts to keep Moore's law going, to get "More Moore". Today's chip designs are hitting fundamental limits: heat and quantum physics. Put more components in the same space and they must get smaller. Much smaller. Each component generates heat. Packing enough components together makes cooling too expensive. By the early 2020's computer chip components will hit ultimate limits, at 10 atoms wide they will belong to the uncertain world of quantum physics. "Moore's Law" is ending. 

When computers were no longer speeding up quickly enough programmers had to start using divide-and-conquer algorithms. Performance increased but at a price. Now instead of programming everything on one computer, programmers had to break problems into many pieces running on separate computers and then merge the results into the final answer.


Let's say you're given the text:

	The quick brown fox jumped over the lazy dog.
	Petunias pickled under the noon sun.

and told to count the number of times each word occurs. Your list ends up looking something like this:

| Word(s)                                                              | Count  |
|----------------------------------------------------------------------|--------|
| the                                                                  | 3      |                                                        
| brown,dog,fox,jumped,lazy,noon,over,petunias,pickled,quick,sun,under | 1 each |

Each word is examined in turn one after another. If a computer uses this approach, faster counting requires a faster computer. 

Turns our there's another way. Break the text into two lines:

	The quick brown fox jumped over the lazy dog.
and

	Petunias pickled under the noon sun.

Calculate the word counts separately:

| the | brown | dog |fox | jumped | lazy | over | quick 
|-----|---|---|--|---|---|---|---|
| 2 | 1 | 1 | 1 | 1 | 1 | 1 | 1

and

| the | noon |petunias | pickled | sun | under |
|-----|------|----|---------|------|----| 
|  1  |  1 | 1 | 1 | 1 | 1 

 Add the two tally sheets together to get the final result

| Word(s)                                                              | Count  |
|----------------------------------------------------------------------|--------|
| the                                                                  | 3      |                                                        
| brown,dog,fox,jumped,lazy,noon,over,petunias,pickled,quick,sun,under | 1 each |

Each line is examined independently. If a computer uses this approach, faster counting is possible with more computers.

The second algorithm is map-reduce. First we break the problem up into bite-sized chunks, in our example we split the text into separate lines. Next we **map** each chunk to a partial answer, in our example we count the words in each line. Finally we combine the partial answers to get the final answer, in our example adding up the counts on both tally sheets **reducing** from two tally sheets to one with the total word counts.


Using multiple computers -- Google's original map-reduce system ran on 1800 networked PC's -- introduced a new problem. The problem with all those machines was the near certainty that some of them would break. Consider flipping a fair coin. Half the time it's heads and half the time tails. You will on average get two heads in a row one quarter of the time. Three heads one eighth of the time and so on. Let's say each PC works 99.99% of the time. Pretty good. The odds that 1800 99.99% reliable machines work without any failures is around 84%.  

We will need to extend our example if we want to account for unreliable calculators. Google solved the reliability issue by recalculating failed work and so will we. Imagine four somewhat unreliable friends: Larry, Moe, Curly and Shemp working the word counting problem together. Moe organizes their efforts. The others do the work. He takes the paper and cuts it into two strips, one for each line, giving one to Curly and the other to Larry. Curly finishes his strip and puts his results on the table along with his paper strip. Larry gets bored, drops his strip on the table, and walks away. How does Moe finish the task? He doesn't have an easy way to tell which line Curly completed. And before you say just compare the words on the sheet with the two strips imagine there were a thousand strips. The solution is to for Moe to add an id to each strip

1

	The quick brown fox jumped over the lazy dog.

2

	Petunias pickled under the noon sun.

and record who he gave each strip to as well as whether it is complete.

**Mapping**

| Line | Who | Status |
|------|-----|----------------|
|  1   |  Larry | Walked Away!#&%$
|  2   |  Curly | Done :-)

Now when Larry walks away Moe can hand strip one to Shemp and complete the task.

A similar approach works for tracking who is combining the partial results into the final one.

# Technical Details #

Time to get away from The Three Stooges&reg;. Let's start with the simplifying assumptions. Map-Reduce transforms Maps with keys not simple strings. Reduce transformations do not always result in fewer elements in the output set. The Moore's law discussion abstracted computer networks, chips, cores and threads into "computers" they are separate concepts.

Map-Reduce chunks input data and maps the chunks to a `Map<key,value>` whose keys are in the input domain. The **map** stage transforms each input map entry into intermediate map entries whose keys are in the solution domain. The **reduce** stage groups intermediate map entries by key and processes each group yielding 0, 1 or more result entries for each key. 

The word count example assumes a single document. The input key is the line # and the value is the line text. The words are the intermediate and result keys. The word-counts per line are the intermediate values. The result values are the number of times that word appears in the input document.
  
The discussion of Moore's law ignored a few technical details. Components were doubling about every two years from 1975 until recently but performance doubled more like once every 18 months. Today's programmers deal with multi-core processors each supporting multiple threads. Logically I collapsed that detail into computers as in things capable of running independent calculations -- if we ignore thread switching and resource contention.

The demos substitute threads for machines. Since threads are reliable an explicit failure recovery mechanism is not required. The system can block until all lines are mapped before proceeding to the reduce stage for the final answer.

![Activity Diagram Map Reduce Demo](https://i.imgur.com/IRHfwTZ.png)



  
