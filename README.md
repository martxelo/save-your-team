# Save your team

This notebook calculates the minimum points your team need to get for maintaining the category. In spanish football league it is said that 41 points is enough, but I wanted to know if thats true.

## The league

Twenty teams face each other during the season. Each team plays 38 matches and gets 3 points for a win, 1 point for a tie and 0 points for a loss. At the end of the season the three teams with the lowest performance fall to a lower division for the next year.

## The problem

There are many different possible results for the whole season. Ten matches per week, thirty eight weeks and three results (the first team wins, the second wins or they tie). So the number of different seasons is:

$$
seasons = 3^{10\times38} = 3^{380} \approx 2.02\times10^{181}
$$

So many... but we can reduce the number. A huge number of seasons are equivalent for our purpose. For example, the first team can have 80 points and the second 77. But one change and the result is 78-78. For the team that is going to a lower division those results are the same.

And there are many simmetries because we don't care if team called *Tigers of the City* wins the league or losses all the matches. We only want to know the maximum points the team that ends the league in the position 17 can have.

## The solution

Linear programming will save us much time. We will actually use mixed integer programming with [PuLP](https://coin-or.github.io/pulp/) library.

So we need to maximize the points for the 17th team at the end of the season. A variable $p_{t}, t\in \left[1,20\right]$ will keep the points of all the teams. And the problem becomes:

$$
\max \hspace{5mm} p_{17}
$$

Quite simple. Now we need to create a variable for storing the result of the matches for the season. It will have three indices: first team, second team, result. So it is like a binary tensor of shape (20, 20, 3). The third index is like a onehot encoding for the result. If index 1 is 1 then the first team won, if index 2 is 1 the it is a tie, and if index 3 is 1 then the second team won the game. Examples:

$$
\begin{aligned}
M_{3,6,1} &= 1 \implies \text{team 3 won}\\
M_{3,6,2} &= 1 \implies \text{it's a tie}\\
M_{3,6,3} &= 1 \implies \text{team 6 won}
\end{aligned}
$$

> **_NOTE:_** there are some variables that won't be used. A team cannot play against itself so $M_{i,i,r} \forall i \in \left[1,20\right]$ is unused.

Now we only have to create the constraints. The first constraint says that only one result per match is possible:

$$
\sum_{r}M_{i,j,r} = 1 \hspace{3mm} \forall i,j
$$

The second constraint will sum the points of every team and every match:

$$
p_{i} = 3\sum_{j\ne i}M_{i,j,1} +
        1\sum_{j\ne i}M_{i,j,2} +
        1\sum_{j\ne i}M_{j,i,2} +
        3\sum_{j\ne i}M_{j,i,3}
$$

That is, 3 points for matches won in your stadium, 1 point for matches tied in your stadium, 1 point for matches tied in opponent's stadium and 3 points for matches won in opponent's stadium.

And the third constraint will keep the points sorted. First team must have at least the same points than the second, and so on:

$$
p_{i} \ge p_{i+1} \hspace{3mm}\forall i \in \left[1, 19\right]
$$

It is quite easy to code this with PuLP.

## The result

For maximizing the points for team 17 the season is really odd. This is the table with the results:

|position|points|wins|ties|losses|
|---|---|---|---|---|
|1|63|21|0|17|
|2|63|21|0|17|
|3|63|21|0|17|
|4|63|21|0|17|
|5|63|21|0|17|
|6|63|21|0|17|
|7|63|21|0|17|
|8|63|21|0|17|
|9|63|21|0|17|
|10|63|21|0|17|
|11|63|21|0|17|
|12|63|21|0|17|
|13|63|21|0|17|
|14|63|21|0|17|
|15|63|21|0|17|
|16|63|21|0|17|
|17|63|21|0|17|
|18|63|21|0|17|
|19|2|0|2|36|
|20|2|0|2|36|

So with 63 points your team is in danger. For being sure you need 64 points.