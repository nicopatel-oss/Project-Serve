# Project-Serve

From in-play win probability to tournament simulations to live betting markets, almost every tennis model assumes points are IID. It’s a convenient assumption, but an odd one for a sport where players, commentators and fans all constantly talk about momentum. So, I created this project to answer the following question. Conditional on the server, does the probability of winning a point on serve depend on whether the server won the previous point? The naive answer is yes. After accounting for confounding, the answer is no. This is the tennis version of the hot hand question in basketball, where the original finding of no effect was itself overturned by Miller and Sanjurjo in 2018 on statistical grounds. 

I used Jeff Sackmann’s ‘charting-m-points-2020s’ dataset, which includes point-by-point data from roughly 2000 professional men’s matches between 2020 and 2026. The data was mostly clean, but after looking at the dataset thoroughly, I noticed 3 issues. The first being a dtype anomaly in the games-won column led me to a Montemar Challenger match (Passaro vs Budkov Kjaer) where the counter reset mid-set, corrupting every row after it, so I dropped the match. Cross-checking score states against their possible paths revealed impossible transitions in two matches (Hurkacz vs Kyrgios, Davidovich Fokina vs Jaziri). This occurred because every point was recorded twice. Instead of removing the matches, I removed the duplicated rows. Lastly, I removed all tiebreak games. In a tiebreak, the server changes within a single game, so “the previous point on the same player’s serve” is not well defined, and the lag structure which the analysis relies on will not work. I identified tiebreaks by detecting games where the server changes mid-game instead of being encoded by the scoring rules, which avoids assumptions about format variations. 

The first quantities I computed were two conditional probabilities: P(server wins | server won previous point) and P(server wins | server loses previous point). Computing these requires dropping the first row of every game, since the previous point was served by the opponent. After a won point, the server wins 64.92% of the time (n = 269,319) and after a lost point, 63.15% (n = 171,076). The difference is 1.8 percentage points. The standard error of that difference is 0.0015, giving z ≈ 12 and an extremely small p-value (but with 440,000 points, significance is almost guaranteed for any non-zero effect, so the magnitude matters more than the test).

Taking this at face value, this is strong evidence against independence. It is also exactly what confounding by player quality would produce. It is a fact that Servers differ largely in quality. A strong server might hold 75% of points, but a weaker server may hold 58% of points. Conditioning on “the server won the previous point” will always favour stronger servers, because better servers are more likely to win. This means that the comparison compares a better server sample against a worse server example, and the positive association appears even if every individual player’s points are perfectly independent.

To resolve the issue of player-quality confounding, I conditioned on score state. This removes the confounding, as every path to a given score state (a,b) contains exactly a wins and b losses. Therefore, under independence regardless of server quality p, every sequence has probability p^a(1-p)^b. The set of paths arriving via a won previous point and the set arriving via the lost one are equally likely for any p, meaning both groups contain the same mix of server quality. Any difference that survives cannot be linked to the heterogeneity between players. 

Before conditioning on score state, my initial idea to eliminate player-quality confounding was a permutation test. The idea was to shuffle point outcomes within each service game while keeping the server, the opponent, the surface and the match situation fixed. This way, any association that survives the shuffle is real sequence dependence rather than differences between players.

The test failed, and the way it failed was informative. The null distribution centred on +0.057 rather than zero, and the observed statistic of 0.0177 sat roughly 32 null standard deviations below the centre. An observed value that far outside its own null usually means the null is misspecified rather than the effect being enormous.

There was also one fundamental problem. Tennis games have a stopping rule, a game ends the instant someone reaches the winning point, so the last point is always won by game’s winner. Therefore, no valid sequence contains a winning position before the end. Shuffling within a game breaks both constraints and generates games that should have already finished. Those sequences are not exchangeable with the real ones, so the null distribution is not a fair comparison.

Conditioning on score state avoids the problem entirely, since it compares points at identical positions in the game rather than attempting to simulate alternative games.

While conditioning on score state, I skipped single-path score states, as they carry no information, leaving eight score states. For each, I computed the difference in server winning win rate between points following a won point and points following a lost one. I then pooled them using inverse variance weights. 

<p align="center">
  <img width="500" alt="Table: Per score state table" src="https://github.com/user-attachments/assets/37ccf52b-834c-409b-887d-b503a693b2ae" />
  <br>
  <em>Table 1: Per score state table.</em>
</p>


The pooled difference is -0.29 percentage points, with a 95% CI (-0.72, +0.14). A Mantel-Haenszel odds ratio for the same score states gives 0.987 with a 95% CI (0.968,1.006) and p-value = 0.17, agreeing closely with the risk-difference estimate. A test of homogeneity across score states returns a p-value = 0.07, so there is no strong evidence to suggest that the effect varies by score state.

The naive result gave us a difference of +1.77 percentage points. Conditioning on the score state contradicts this and tips it slightly negative.

With over 207,000 points, the analysis had 80% power to detect a difference of 0.6 percentage points. The observed estimate is smaller than that, and the confidence interval excludes effects larger than roughly 0.7 percentage points in either direction. This is a bounded null, not merely an absence of evidence. 

There are a few limitations to this project that should be kept in mind. The first being that the data was crowdsourced through the Match Charting Project, so the coverage is driven by what volunteers chose to chart and is likely skewed towards notable players and memorable matches. The data consists of only men’s matches and is from the 2020s. The result does not represent the women’s tour or earlier eras in tennis, which may have a different conclusion.

More importantly, the tests depended on the immediately preceding point only. Momentum usually runs over long stretches (a run of lost points, a broken serve, a set slipping away) and a null result at lag 1 does not rule that out. Extending the analysis to longer lags, or to game and set level outcomes, is the obvious next step. 

Finally, the heterogeneity test across different score states return a p-value = 0.07, which does not establish that the effect is constant. With only 8 score states, the test has limited power, so mild variation between score states would likely go undetected. 
 
## Data
This project uses data from [The Tennis Abstract Match Charting Project](https://github.com/JeffSackmann/tennis_MatchChartingProject)
by Jeff Sackmann, licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
The data is not included in this repository.
 

   


 
