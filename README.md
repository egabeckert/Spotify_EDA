# Spotify Track Popularity: Exploratory Data Analysis

## What this is
Exploratory data analysis on 32,833 rows of Spotify track data, completed for my first course in the Master of Data Science program at the University of Pittsburgh. The question: which audio features and categorical attributes actually line up with track popularity, and which ones look like they matter but don't?

The goal wasn't to build a model yet. It was to understand the data well enough to know what a model should and shouldn't lean on, and to make deliberate decisions about the messy parts instead of defaulting to the standard cleanup.

## Data
- Source: [TidyTuesday, January 2020](https://github.com/rfordatascience/tidytuesday/tree/master/data/2020/2020-01-21), loaded directly from the raw CSV in the notebook
- 32,833 rows, where each row is a track on a specific playlist, not a unique song
- Outcome variable: `track_popularity` (0 to 100)

## Key Decisions

**Transforming popularity.** Popularity is bounded between 0 and 100, which squeezes values near the edges. I converted it to a fraction and applied a logit transform, first shifting exact 0 and 100 values by 0.1 so the transform doesn't produce infinite values. The result is roughly symmetric, from about -6.9 to 6.9.

**Duplicates.** Songs appear on multiple playlists, so the same `track_id` shows up in multiple rows. Before deciding what to do, I checked that popularity stays constant within each `track_id`, which means the duplication doesn't distort the outcome. Instead of dropping the duplicates, I kept them and engineered `num_playlists`: the number of distinct playlists each track appears in. One tradeoff to keep in mind: genre-based charts count playlist-genre entries, not strictly unique songs.

**Variable selection.** Identifier columns were excluded from analysis. `mode` and `key` are stored as integers but treated as categorical, since one is binary (major/minor) and the other is one of 12 musical keys. I kept `playlist_genre`, `mode`, `key`, `valence`, `energy`, and `num_playlists`. I dropped `loudness` because it likely overlaps with `energy`, and `tempo` because 17,684 unique values made it too granular to use without binning.

**Visualizing at scale.** Regular scatterplots were unreadable at this row count, so I switched to hexbin density plots. Splitting those by genre then showed clustering that the combined view was hiding.

## Key Findings

- **No single variable cleanly separates popular and unpopular tracks.** Popular songs are spread across the full range of energy and valence, so audio features on their own are probably weak predictors.
- **`num_playlists` was the clearest signal in the dataset.** Popularity rises with playlist count early on, then levels off. The most popular songs aren't at the maximum of 8 playlists, so playlist reach looks more like a popularity floor than something that drives the very top performers. The duplicate rows turned out to be useful rather than a problem.
- **Genre explains a song's sound more than its popularity.** Energy hotspots differ by genre (EDM and rock cluster around 0.8 to 1.0, rap and R&B spread across 0.4 to 0.8), and rock leans major about 70% of the time. Median popularity, though, is about the same across genres.
- **Mode doesn't move popularity,** overall or within any genre. One odd quirk: minor-key tracks skewed slightly higher in valence (around 0.6) than major-key tracks (around 0.4), against the usual assumption that major means happier.
- **Rock was the only genre with no high-popularity outliers,** which may mean it doesn't produce the same viral hits other genres do.

## Limitations / What I Learned

This analysis is visual and exploratory. There's no statistical testing or modeling, so the patterns above are observations to test, not confirmed effects. There are also relatively few high-popularity tracks, which limits what can be said about the very top performers.

The biggest takeaway was that a data-quality issue can turn into a signal if you investigate it before "fixing" it. Dropping the duplicates would have thrown away the most informative feature in the dataset. I also learned that a confident "no" is a useful result. Ruling out mode as a driver of popularity narrows what a model needs to focus on.

With more time, I'd build a model combining several features, test whether the `num_playlists` relationship holds statistically, and rerun the key comparisons on a deduplicated, one-row-per-song version as a sensitivity check.

## Stack
```
pandas, numpy, matplotlib, seaborn, Jupyter
```

## How to Run
```bash
git clone https://github.com/egabeckert/Spotify_EDA.git
cd Spotify_EDA
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook egab_Spotify_EDA.ipynb
```
The data loads directly from the TidyTuesday URL, so no local download is needed.
