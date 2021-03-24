Are Roger and Serena the all-time best tennis players?
================
Your Teacher Today: Bobby Stuijfzand

## Before we begin

### Some steps to follow if you want to code along

1.  Make sure RStudio is open
2.  Create a new RStudio project in a new directory (name does not
    matter)
3.  Create a folder in the RStudio project directory called `data`
4.  Inside the main folder (not inside `data`), add the learner
    Rmarkdown file of this learner lab (`learner.Rmd`). You can find it
    here: <https://github.com/BobbyGlennS/20210326-learnerlab>

On top of what you have installed for FDS, there are some additional
packages that you need. Don’t forget to run the following chunk in the
console before we begin.

``` r
install.packages("here")
install.packages("vroom")
```

### Beginning our R code

The first step in any R script is to load the relevant packages. We will
load the `{tidyverse}`, `{here}`, and `{vroom}`, so you need to execute
the following chunk:

``` r
library(here)
library(tidyverse)
library(vroom)
```

## Defining the question

In this webinar we will use data from the WTP and ATP to answer the
question:

***Who are the best tennis players of all-time?***

You might wonder, aren’t we asking whether Roger and Serena are the best
players? Not really. That was just a journalist trick, a headline to
lure people into this webinar. I prefer to ask a slightly more open
question, a question that isn’t simply answered with yes or no, as with
a similar amount of data analysis effort we will learn a lot more!

To answer such a question, we need to put some definitions and
operationalisations in place. After all, we can mean different things
with what constitutes as the all-time best tennis player.

### The scope

Let’s first work out the exact scope of the question.

We will limit this question to each gender separately - we are not going
to compare between genders. I use this scope as genders do not compete
against each other, so there is no point in making a comparison.

We will further look at who won the grand slam single tournaments. We
could be more comprehensive, by incorporating rankings for example, but
it would not be feasible to do this within the time frame.

We’ve narrowed the scope of the question down to the resources we have
available, both data (more on that in a minute) and timewise.

This is a common challenge in data science.

### Formulate some sub questions

We will use two questions to get some insight into our main question.

-   Who won the most grand slam tournaments (single player)?
-   Who won the most grand slam tournaments consecutively?

After we have found answer to these questions, we can combine this
information to give a comprehensive answer to our main question.

## The data

### Credit where credit is due

The dataset we will be using is carefully assembled by [Jeff
Sackmann](https://github.com/JeffSackmann) and is freely available on
the internet in the form of github repositories, links for which you can
find here:

-   <https://github.com/JeffSackmann/tennis_atp>
-   <https://github.com/JeffSackmann/tennis_wta>

To code along with this webinar you will have to download these
repositories to your laptop and store them as folders under the `data`
folder in your RStudio project. I will show you how to do this in the
webinar.

### Exploring the data

Have a look at the repositories. There are many files. The *csv* files
beginning with *“wta\_matches”* or *“atp\_matches”* and then a year are
the ones we want to use as they contain the information on who won a
singles grand slam tournament.

### Load the data

How do you load many files at once?

#### Get the filenames

We start with listing the files we are interested in with
`list.files()`. We will do this separately for the wta (women’s) and atp
(men’s) data.

``` r
data_paths_wta <- list.files(
  path = here("data", "tennis_wta-master"), 
  pattern = "^wta_matches_\\d.+csv"
)

data_paths_atp <- list.files(
  path = here("data", "tennis_atp-master"), 
  pattern = "^atp_matches_\\d.+csv"
)
```

You can see that the `list.files()` function takes two arguments:

`path` defines the path to where we want to see the files from. We use
the function `here()` to define the subfolders relative to the RStudio
project directory that get us to the right place. This trick will only
work if you are working from your RStudio project.

`pattern` defines a pattern in the filenames that we are looking for.
The code here is a *regular expression*. It is beyond the scope of this
webinar to explain how they work, but what it will do is keep only the
filenames we want, the ones containing singles matches.

#### Get the data itself

Then we load in the data in one go with `vroom()`. `vroom()` is a nice
little function that can read many datafiles, and if they have the same
structure, it will put all data underneath each other. To know from what
file each row in the data came, we add an `id` column called `file`.
`vroom()` will populate this column with the filename and path for the
relevant data.

``` r
data_wta <- vroom::vroom(
  here::here("data", "tennis_wta-master", data_paths_wta), 
  id = "file"
) 

data_atp <- vroom::vroom(
  here::here("data", "tennis_atp-master", data_paths_atp), 
  id = "file"
) 
```

Let’s now admire our data:

``` r
head(data_wta) %>% knitr::kable()
```

| file                                                                                                              | tourney\_id | tourney\_name  | surface | draw\_size | tourney\_level | tourney\_date | match\_num | winner\_id | winner\_seed | winner\_entry | winner\_name       | winner\_hand | winner\_ht | winner\_ioc | winner\_age | loser\_id | loser\_seed | loser\_entry | loser\_name        | loser\_hand | loser\_ht | loser\_ioc | loser\_age | score        | best\_of | round | minutes | w\_ace | w\_df | w\_svpt | w\_1stIn | w\_1stWon | w\_2ndWon | w\_SvGms | w\_bpSaved | w\_bpFaced | l\_ace | l\_df | l\_svpt | l\_1stIn | l\_1stWon | l\_2ndWon | l\_SvGms | l\_bpSaved | l\_bpFaced | winner\_rank | winner\_rank\_points | loser\_rank | loser\_rank\_points |
|:------------------------------------------------------------------------------------------------------------------|:------------|:---------------|:--------|-----------:|:---------------|--------------:|-----------:|-----------:|-------------:|:--------------|:-------------------|:-------------|-----------:|:------------|------------:|----------:|------------:|:-------------|:-------------------|:------------|----------:|:-----------|-----------:|:-------------|---------:|:------|--------:|-------:|------:|--------:|---------:|----------:|----------:|---------:|-----------:|-----------:|-------:|------:|--------:|---------:|----------:|----------:|---------:|-----------:|-----------:|-------------:|---------------------:|------------:|--------------------:|
| /Users/bstujifzand/Documents/EXTS/workshops/20210326\_learner\_lab/data/tennis\_wta-master/wta\_matches\_1949.csv | 1949-1001   | Port Elizabeth | Clay    |         NA | W              |      19481230 |          1 |     228989 |           NA | NA            | Toodles Watermayer | U            |         NA | RSA         |    27.29363 |    229256 |          NA | NA           | Hazel Redick Smith | R           |        NA | RSA        |         NA | 6-0 6-4      |        3 | F     |      NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |           NA |                   NA |          NA |                  NA |
| /Users/bstujifzand/Documents/EXTS/workshops/20210326\_learner\_lab/data/tennis\_wta-master/wta\_matches\_1949.csv | 1949-1002   | Cape Town      | Clay    |         NA | W              |      19481230 |          1 |     230891 |           NA | NA            | Mary Muller        | U            |         NA | RSA         |          NA |    228999 |          NA | NA           | Penelope Pentelow  | U           |        NA | RSA        |         NA | 2-6 6-4 6-1  |        3 | F     |      NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |           NA |                   NA |          NA |                  NA |
| /Users/bstujifzand/Documents/EXTS/workshops/20210326\_learner\_lab/data/tennis\_wta-master/wta\_matches\_1949.csv | 1949-1003   | Adelaide       | Grass   |         NA | W              |      19481227 |          1 |     216619 |           NA | NA            | Doris Hart         | R            |         NA | USA         |    23.52088 |    232614 |          NA | NA           | G Mason            | U           |        NA | AUS        |         NA | 6-0 6-2      |        3 | R16   |      NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |           NA |                   NA |          NA |                  NA |
| /Users/bstujifzand/Documents/EXTS/workshops/20210326\_learner\_lab/data/tennis\_wta-master/wta\_matches\_1949.csv | 1949-1003   | Adelaide       | Grass   |         NA | W              |      19481227 |          2 |     224651 |           NA | NA            | Nell Hopman        | U            |         NA | AUS         |    39.80287 |    230361 |          NA | NA           | Clare Proctor      | U           |        NA | AUS        |         NA | 8-6 2-6 10-8 |        3 | R16   |      NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |           NA |                   NA |          NA |                  NA |
| /Users/bstujifzand/Documents/EXTS/workshops/20210326\_learner\_lab/data/tennis\_wta-master/wta\_matches\_1949.csv | 1949-1003   | Adelaide       | Grass   |         NA | W              |      19481227 |          3 |     227740 |           NA | NA            | Marie Martin       | U            |         NA | AUS         |          NA |    232615 |          NA | NA           | L Nottage          | U           |        NA | AUS        |         NA | 6-2 6-2      |        3 | R16   |      NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |           NA |                   NA |          NA |                  NA |
| /Users/bstujifzand/Documents/EXTS/workshops/20210326\_learner\_lab/data/tennis\_wta-master/wta\_matches\_1949.csv | 1949-1003   | Adelaide       | Grass   |         NA | W              |      19481227 |          4 |     216626 |           NA | NA            | Thelma Long        | U            |         NA | AUS         |    30.20397 |    230336 |          NA | NA           | Helen Angwin       | U           |        NA | AUS        |         NA | 6-2 6-2      |        3 | R16   |      NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |     NA |    NA |      NA |       NA |        NA |        NA |       NA |         NA |         NA |           NA |                   NA |          NA |                  NA |

It’s a bit wide thanks to the `file` column’s content, but using
`kable()` makes that we still have a nice format to explore this data,
though I wouldn’t suggest keeping it this way in a final report.

#### EXERCISE

Let’s look at some columns we might be interested in using `select()`.
What columns might you include in your report to communicate a general
sense about this data?

``` r
data_wta %>% 
  sample_n(20) %>% 
  select(tourney_name, round, winner_name) %>% 
  knitr::kable()
```

| tourney\_name              | round | winner\_name         |
|:---------------------------|:------|:---------------------|
| Warrnambool 1 10K          | R32   | Renee Reid           |
| Harpenden                  | QF    | Rita Jarvis          |
| Berlin                     | R16   | Elena Dementieva     |
| US National Championships  | R16   | Gwyn Thomas          |
| Quebec City                | SF    | Tamira Paszek        |
| US Open                    | R64   | Stephanie Rehe       |
| US Open                    | R128  | Chanelle Scheepers   |
| Athens                     | R16   | Julie Halard Decugis |
| Sebring                    | R64   | Mimi Wikstedt        |
| Sydney                     | R32   | Anne Smith           |
| Brighton                   | R32   | Sandra Dopfer        |
| Surabaya                   | R16   | Fang Li              |
| Miami                      | R32   | Simona Halep         |
| Toronto                    | QF    | Virginia Ruzici      |
| Fed Cup G1 PO: KAZ vs TPE  | RR    | Hsiao Han Chao       |
| Beckenham                  | R16   | Peggy Dawson Scott   |
| Fed Cup G2 RRB: TUN vs CYP | RR    | Selima Sfar          |
| Fed Cup G1 PPO: ARG vs COL | RR    | Paula Ormaechea      |
| London Indoors             | R32   | Georgie Woodgate     |
| Australian Open            | R32   | Mary Joe Fernandez   |

## Preparing the data

### Creating the relevant subset

The data is very large, with 197083 and 178077 rows for women and men
respectively. We want to avoid including all this data all the time, so
let’s start with filtering the data that we want to keep.

We want to only keep Grand Slam data, and only data on the final round.

#### EXERCISE

Using the columns `tourney_level` and `round`, can you work out how to
filter the data so that we only keep a subset that satisfies the
conditions above?

``` r
data_wta_g_winners <- data_wta %>% 
  filter(tourney_level == "G", round == "F")

data_atp_g_winners <- data_atp %>% 
  filter(tourney_level == "G", round == "F")
```

*Note:* Normally I would do some exploration of the full data first, or
at least read the documentation. As we are collaboratively working here
and these operations maybe slow on some computers, I chose here to
immediately cut to the chase. It’s a compromise made for this webinar,
not necessarily reflective of the usual pipeline.

### Adding dates

For our second question, where we look at consecutive wins, we need to
have our data chronologically ordered. To ensure this is the case, we
can change the `tourney_date` column into a date type and sort our data.

``` r
data_wta_g_winners <- data_wta_g_winners %>% 
  mutate(date = lubridate::ymd(tourney_date)) %>% 
  arrange(date)

data_atp_g_winners <- data_atp_g_winners %>% 
  mutate(date = lubridate::ymd(tourney_date)) %>% 
  arrange(date)
```

Our data is now ready. We’ve reduced it to containing only grand slam
winners and ordered it by date. We now have two very compact datasets of
288 rows and 51 columns for the women’s data and 212 rows and 51 columns
for the men.

## First question: Who won the most grand slam tournaments (single player)?

#### EXERCISE

What code would you use to get the top 5 winners for each gender?

``` r
data_wta_g_winners %>% 
  count(winner_name, sort = TRUE) %>% 
  top_n(n, n = 5) %>% 
  knitr::kable(caption = "Top 5 Grand Slam Women",
               col.names = c("Athlete", "*n*"))
```

| Athlete             | *n* |
|:--------------------|----:|
| Margaret Court      |  24 |
| Serena Williams     |  23 |
| Steffi Graf         |  22 |
| Chris Evert         |  18 |
| Martina Navratilova |  18 |

Top 5 Grand Slam Women

``` r
data_atp_g_winners %>% 
  count(winner_name, sort = TRUE) %>% 
  top_n(n, n = 5) %>% 
  knitr::kable(caption = "Top 5 Grand Slam Men",
               col.names = c("Athlete", "*n*"))
```

| Athlete        | *n* |
|:---------------|----:|
| Rafael Nadal   |  20 |
| Roger Federer  |  20 |
| Novak Djokovic |  18 |
| Pete Sampras   |  14 |
| Bjorn Borg     |  11 |

Top 5 Grand Slam Men

The results might be a bit surprising. If they do, you may want to
google ‘open era in tennis’ and see if you can work out why these
results are not what you expected them to be.

## Second question: Who won the most grand slam tournaments consecutively?

This question is a bit tougher to answer. We need to keep the following
things in mind:

-   An athlete can win once, then lose, and then win again. We don’t
    want these two wins to count towards the same streak.
-   An athlete can have multiple streaks, so the answer will not be as
    unambiguous as in the previous part.

What we need is some code that checks for each athlete if they won, and
if they did, did they win last time too? And then somehow integrate it
into a comprehensive number.

I will demonstrate an approach in steps for the women. Then you can use
this as an exercise to solve it for the men.

### Step 1: Number each game consecutively

We need to figure out a way where we can compare each game with the
previous one. We can’t use date or year, there are multiple grandslams
in a year, and the interval between them is unequal, making it hard to
work with.

Instead, we will create a game id, numbering each consecutive game. This
approach also has as advantage that if we wanted to answer this question
for a subset of our data, e.g. one type of grand slam tournament, we can
reuse the same code on this subset.

We will use `row_number()` for this.

``` r
data_wta_g_cons_winners <- data_wta_g_winners %>% 
  mutate(game_id = row_number()) 
```

We now have a number for each game:

``` r
data_wta_g_cons_winners %>% 
  head(10) %>% 
  select(tourney_name, date, game_id)
```

    ## # A tibble: 10 x 3
    ##    tourney_name              date       game_id
    ##    <chr>                     <date>       <int>
    ##  1 Australian Championships  1949-01-22       1
    ##  2 Roland Garros             1949-05-18       2
    ##  3 Wimbledon                 1949-06-20       3
    ##  4 US National Championships 1949-08-29       4
    ##  5 Australian Championships  1950-01-20       5
    ##  6 Roland Garros             1950-05-19       6
    ##  7 Wimbledon                 1950-06-26       7
    ##  8 US National Championships 1950-08-27       8
    ##  9 Australian Championships  1951-01-22       9
    ## 10 Roland Garros             1951-05-23      10

### Step 2: Group our data by athlete

We want to evaluate *for each athlete individually* how many games they
won consecutively, so we will group the data by `winner_name` so that
our next steps are operated for each winner separately.

``` r
data_wta_g_cons_winners <- data_wta_g_cons_winners %>% 
  group_by(winner_name)
```

Before we continue, let’s have a look at the data.

``` r
data_wta_g_cons_winners %>% 
  arrange(winner_name) %>% 
  select(winner_name, game_id, date) %>% 
  View()
```

### Step 3: Find the id of the previous game\_id

What we want to know is whether a game was the start of a streak, or
belonged to a streak. We can use `lag()` to find the previous `game_id`
an athlete has won. We use `default = -99` so that if there is no
previous game, we still have a value to work with. One that won’t mess
up our next calculations.

``` r
data_wta_g_cons_winners <- data_wta_g_cons_winners %>% 
  mutate(previous_game = lag(game_id, default = -99))
```

Let’s examine the data again:

``` r
data_wta_g_cons_winners %>% 
  arrange(winner_name) %>% 
  select(winner_name, game_id, previous_game, date) %>% 
  View()
```

### Step 4: Is the current id the start of something new?

We can now compare the `game_id` with that of the `previous_game`. If
the difference between the two is **not** 1, it means that the current
id was not preceded by a win in the tournament that came right before
this one. In other words: it is the start of a new streak.

``` r
data_wta_g_cons_winners <- data_wta_g_cons_winners %>% 
  mutate(streak_begin = game_id - previous_game != 1)
```

Let’s examine the data again:

``` r
data_wta_g_cons_winners %>% 
  arrange(winner_name) %>% 
  select(winner_name, game_id, previous_game, streak_begin) %>% 
  View()
```

### Intermezzo: the cumulative sum

For the next step, we are going to process the `streak_begin` variable
in a way that we will get a `streak_id` for each separate streak. In
order to do so we can use a function called `cumsum()` which calculates
the cumulative sum for a variable.

What is the cumulative sum?

Let’s demonstrate.

We have a vector containing the values `1, 2, 3, 4`.

Calculating the cumulative sum gives:

``` r
cumsum(c(1,2,3,4))
```

    ## [1]  1  3  6 10

The cumulative sum returns a vector of the same length, that contains
for each element the sum of the original vector *up until that point*.

How is this useful?

Consider that `TRUE` and `FALSE` are represented by `1` and `0` in R. A
cumulative sum on such a logical vector would look like this:

``` r
cumsum(c(FALSE, TRUE, FALSE, FALSE, TRUE))
```

    ## [1] 0 1 1 1 2

You can see that only once a `TRUE` appears will the cumulative sum
increment.

Now recall that in our `streak_begin` variable, we have a `TRUE` for
every start of a streak.

Let’s look at the data again:

``` r
data_wta_g_cons_winners %>% 
  arrange(winner_name) %>% 
  select(winner_name, game_id, previous_game, streak_begin) %>% 
  View()
```

### Step 5: Get an id for each streak

If we run `cumsum()` on `streak_begin`, what we will get is a number
that increments with every new beginning. In other words: we have a
unique number for each streak 🎉

``` r
data_wta_g_cons_winners <- data_wta_g_cons_winners %>% 
  mutate(streak_id = cumsum(streak_begin))
```

Let’s admire our result.

``` r
data_wta_g_cons_winners %>% 
  arrange(winner_name) %>% 
  select(winner_name, game_id, previous_game, streak_begin, streak_id) %>% 
  View()
```

### Step 6: Let’s count the streaks!

Now all that’s left is counting up the streak ids to understand how many
rows belonged to each streak, and therefore, how long each streak was!

``` r
data_wta_g_cons_winners_result <- data_wta_g_cons_winners %>% 
  count(streak_id, sort=TRUE)
```

Let’s admire the result

``` r
data_wta_g_cons_winners_result %>% 
  head(20) %>% 
  knitr::kable()
```

| winner\_name        | streak\_id |   n |
|:--------------------|-----------:|----:|
| Margaret Court      |         10 |   6 |
| Martina Navratilova |          4 |   6 |
| Maureen Connolly    |          2 |   6 |
| Steffi Graf         |          2 |   5 |
| Serena Williams     |          2 |   4 |
| Serena Williams     |         13 |   4 |
| Steffi Graf         |          6 |   4 |
| Billie Jean King    |          2 |   3 |
| Billie Jean King    |          5 |   3 |
| Chris Evert         |         11 |   3 |
| Margaret Court      |          8 |   3 |
| Martina Hingis      |          2 |   3 |
| Martina Navratilova |          3 |   3 |
| Monica Seles        |          3 |   3 |
| Shirley Fry         |          2 |   3 |
| Steffi Graf         |          3 |   3 |
| Steffi Graf         |          7 |   3 |
| Steffi Graf         |          8 |   3 |
| Althea Gibson       |          2 |   2 |
| Althea Gibson       |          3 |   2 |

As we stored the results in a separate tibble, We can now also go back
to the original tibble to examine some individual streaks.

For example Serena Williams longest streak:

``` r
data_wta_g_cons_winners %>% 
  filter(streak_id == 13) %>% 
  select(winner_name, date, tourney_name)
```

    ## # A tibble: 5 x 3
    ## # Groups:   winner_name [2]
    ##   winner_name     date       tourney_name   
    ##   <chr>           <date>     <chr>          
    ## 1 Chris Evert     1986-05-26 Roland Garros  
    ## 2 Serena Williams 2014-08-25 US Open        
    ## 3 Serena Williams 2015-01-19 Australian Open
    ## 4 Serena Williams 2015-05-25 Roland Garros  
    ## 5 Serena Williams 2015-06-29 Wimbledon

Hey! Why does Chris Evert appear? This is because the `streak_id` was
created on the grouped tibble. I could have ungrouped before counting
the streak ids, but I would have lost the names, so I didn’t do that.
`count()` does take into account existing grouping when computing
counts, so that decision did not affect our results.

## And now for the men…

#### EXERCISE:

Can you answer the second question for the men?

## Open era tennis results

#### BONUS:

What would the results look like if we account for the switch to
allowing professionals to compete in grand slams?

Hint: the open era, as it was called, started in 1968.
