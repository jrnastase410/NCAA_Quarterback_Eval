# NCAA-Quarterback-Eval

Full Results (2013-2021) Here: https://docs.google.com/spreadsheets/d/1gW_2UKQDlHQMA9MUbOvTRwH4sRUXjBAEOrUl1y1gWpw/edit?usp=sharing

A Bayesian-inspired methodology for grading NCAA quarterbacks. Takes into account per-play impact, recruiting ratings, strength of schedule, etc.

Who is a better quarterback?
QB A: 22 yards per attempt, perfect QB rating, perfect TD/INT ratio.
QB B: 8 yards per attempt, solid but not spectacular rating, 2/1 TD/INT ratio.

To this point, boiling down these performances to a single number suffers from a serious flaw. How many passes did each throw? How many plays were they involved in? In other words, what was the context, and has the performance been sustained long enough to be confident in the projection of it going forward? The easiest solution to this is to simply apply filters, perhaps we only look at QBs that have thrown 100+ passes. This however doesn't allow investigations into the questions fans really care about. Is the backup QB better, or isn't he? Hard to tell if he hasn't yet thrown a pass.

This algorithm is designed to address this flaw, by assuming an expectation for each player, based on playing time and recruiting evaluations, and allow each player to move his rating away from this expecation as his performance dictates. To put this into action, we'll take 5 steps.

1. Compute the primary metric. This algorithm is primarily concerned with rightly estimating a player's skill, _given_ a metric that is a good indicator of performance on a game-to-game basis. It is an attempt to aggregate, while accounting for uncertainty, but doesn't provide the initial metric that will be aggregated. In this project, the metric _ppa_raw_ is designed based on two criteria:

a.	averagePPA.all - PPA (Predicted Points Added) is developed by predicting a team's expected points on the drive at the start of any given play, and then calculating the added expected points on each play. For example, if Team A has the ball, 1st and 10 on their own 20 yard line, you would expect them to score roughly 2.5 points on average. If the quarterback runs for 79 yards to the opponent's 1 yard line, that expectation goes up to 6 points. This play would then be worth 3.5 PPA. 'averagePPA.all' then, is the average of PPA per play that the QB is involved in (rush / sack / pass).
   
b. ppa_exp - This is the PPA you would expect a team to achieve, given the strength of the defense. This is calculated by taking the SP+ defense rating, and running a Ridge regression against PPA data. These criteria combine to make _ppa_raw_ (averagePPA.all - ppa_exp) or (actual_value - predicted_value) to determine the skill of the quarterback.

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////// 

2. Next, in order to aggregate, a weight needs to be assigned. In this project, that will be determined by the number of plays for a QB, and the time passed since the play (i.e. 2021 plays would have a higher weight than 2020). The allocation of these weights can be adjusted under parameters in the notebook. The ppa_raw rating is then multiplied by the weight to create the ppa_wt column.

The other two columns create in this step are 'agg_weight', and 'year_agg_weight'. 'agg_weight' is the cumulative to-date sum weight, where 'year_agg_weight' is the 'agg_weight' scaled to account for the diminishing weights for past years to put the overall ratings on the same playing field.

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////// 

3. Next, we create the agg_rating, or ppa_agg. This simply uses the ppa_wt rating and the agg_weight column to create a rolling weighted average for ppa_raw.

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////// 

4. Now, we implement the bread and butter of this algorithm, the adj_rating, or ppa_adj. Once we have the agg_rating, we have a summary of performance to date, weighted by playing-time, but playing time still isn't _rewarded_, i.e. you could have a player with a through-the-roof agg_rating, simply because he hasn't played enough for that rating to stabilize. The adj_rating takes into account three metrics:

a. agg_rating - The measure of performance to date
b. ppa_pred - This is the predicted ppa_raw value based on a player's recruiting grade, weighted by the number of years since the player was recruited.
c. ppa_rep - This represents the replacement level quarterback. The assumption at play here is that, as time passes and a quarterback does not play, even if he was        heavily recruited out of highschool, the lack of playing-time would indicate a lack of ability, and the recruiting weight would give way to the replacement weight.\
These factors are combined as ppa_adj = (ppa_agg * agg_wt + ppa_pred * pred_wt + ppa_rep * rep_wt) / (agg_wt + pred_wt + rep_wt)

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////// 

5. Finally, we produce the overall rating, or ppa_ovr, or simply ovr. This is the ppa_adj rating, scaled to a normal distribution with mean=50, std=10, borrowing from the MLB scouting (20-80) scale. (See fangraphs explanation here: https://blogs.fangraphs.com/scouting-explained-the-20-80-scouting-scale/)

And that is it! A talent-adjusted, defense-adjusted, playing-time-rewarding system for evaluating quarterbacks, even those that have yet to play a down! See the notebook for results, and in the meantime, your **top 10 QBs in college football as of 10/19/2021...**

1. Matt Corral (Ole Miss) - 74.1
2. Sam Howell (North Carolina) - 67.8
3. Micale Cunningham (Louisville) - 67.3
4. C.J. Stroud (Ohio State) - 66.3
5. Tanner Morgan (Minnesota) - 65.9
6. Adrian Martinez (Nebraska) - 65.4
7. Brock Purdy (Iowa State) - 65.4
8. Bryce Young (Alabama) - 65.4
9. Sean Clifford (Penn State) - 64.7
10. Dorian Thompson-Robinson (UCLA) - 63.7

**And, the top 5 since 2013:**
1. Kyler Murray (Oklahoma) - 89.4
2. Baker Mayfield (Oklahoma) - 86.1
3. Tua Tagovailoa (Alabama) - 84.6
4. Mac Jones (Alabama) - 84.2
5. Joe Burrow (LSU) - 83.1
