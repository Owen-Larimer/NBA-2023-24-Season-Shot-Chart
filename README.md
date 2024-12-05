# NBA-2023-24-Season-Shot-Chart
A detailed look into NBA shooting statistic leaders by zone throughout the 2023-24 NBA season:
[23-24 Shot Chart](https://public.tableau.com/app/profile/owen.larimer/viz/NBA2023-24SeasonShotChart/Dashboard1#1)

### Project Overview

In this project, the goal was to analyze and visualize shooting statistics for the NBA's 2023-2024 season. By looking at these statistics for various zones, we can glean insights into which players are the best offensively from all areas of the court as well as look at each player's individual statistics for each zone. 

### Data Sources

NBA shot data: The primary dataset used in this project is one of my own making, the "NBA_2023_24_shot_data.xlsx" file, containing a row for each player who played during the season, and their **FGM** (Field Goal Makes), **FGA** (Field Goal Attempts), and **FG%** (Field Goal Percentage) from 6 different areas on the court.
- This data was obtained from the official NBA site [https://www.nba.com/stats](https://www.nba.com/stats). API's exist to pull such data from years past and real time, and for those looking to automate projects like this or get cleaner data for a multitude of statistics, that's what I would do; however, it was simple enough for my needs to just extract the data from their page into excel and then practice cleaning it myself.

### Tools

- Excel - Initialized Dataset; Some Cleaning
- Jupyter Notebook (Python and Pandas) - Major Cleaning; Some Analysis and Visualization
- Tableau - Heavy Visualization and Interactables
  
### Data Cleaning/Preparation Phase

In cleaning in Excel, we performed the following:
1. Removed all duplicate values
2. Adjusting column datatypes
3. Ridding stray null values

The majority of cleaning took place in JupyterLab, using Pandas (See: "NBA_Shot_Chart_Cleaning.ipynb" file).
1. Got rid of null values that were hidden in Excel.
2. Converted many columns into numerical ones.
3. Viewed the data around certain conditionals, such as shot attempts.
4. Did some early visualization to get information on the best players in a specific zone -> This would later aid in visualization testing.
5. Initially loading the data into a csv file for use (See: "shot_chart_v1.csv").

After __many__ failed attempts at creating a map inside Tableau, I reloaded my data into Pandas to do some more in depth cleaning:
1. Fixed many player names that contained non-traditional characters not present on standard keyboards.
2. Melted my data in order to later brute force my data into mappable areas.
3. Adjusted column and value names to be more user-friendly and readable.
4. Artificially created coordinate columns that could be used to map data in Tableau
5. Created dataframes specifically for FGM, FGA, and FG%, then joined them all into a master table.
6. Loaded the data into a new csv file (See: "all_shots.csv").

#### Data Cleaning Code Examples:

To view all names that contained special characters using regular expressions:
```python
df[df['Player'].str.contains(r'[^\x00-\x7F]', regex=True)]
```

To manually create coordinates for each zone specific to the image used in background of map:
```python
x_coord_mapping = {
    'Restricted Area': 0,
    'Paint (-Rest)': -10,
    'Mid Range': 10,
    'Left Corner': -235,
    'Right Corner': 235,
    'Above Break': -10
    # Add other shot types if needed
}

y_coord_mapping = {
    'Restricted Area': 0,
    'Paint (-Rest)': 50,
    'Mid Range': 100,
    'Left Corner': 0,
    'Right Corner': 0,
    'Above Break': 250
    # Add other shot types if needed
}

FGM['X Coordinate'] = FGM['Shot Type'].map(x_coord_mapping)
FGM['Y Coordinate'] = FGM['Shot Type'].map(y_coord_mapping)
```
To lengthen my values to allow for mapping in Tableau (Same Process for FGM, FGA, and FG%):
```python
FGM = pd.melt(my_nba, id_vars=['Player', 'Age', 'Team'], value_vars=['Rest_Area_FGM', 'Paint_FGM', 'Mid_FGM', 'L_Corner_3_FGM', 'R_Corner_3_FGM', 'Above_Break_3_FGM'])
```

To merge all three FGM, FGA, and FG% lengthened dataframes:
```python
first_merge = FGM.merge(FGA, on=['Player', 'Age', 'Team', 'Shot Type', 'X Coordinate', 'Y Coordinate'])
second_merge = first_merge.merge(FG_Percentage, on=['Player', 'Age', 'Team', 'Shot Type', 'X Coordinate', 'Y Coordinate'])

order = ['Player', 'Age', 'Team', 'Shot Type', 'FGM', 'FGA', 'FG%', 'X Coordinate', 'Y Coordinate']
all_shots = second_merge[order]
```

### Creating the Map

My initial attempt at mapping the shots in Tableau was rough. My Tableau skills were pretty green and I had never utilized background images as a basis for a scatterplot before. Getting the image onto the actual worksheet was a challenge in itself, as the data _needs_ X and Y coordinates to put the image into a usable worksheet. Otherwise, the background image will just stay as invalid. I was able to create these inside Tableau using a *Calculated Field*, and was able to add the image. However, after a long time messing around in Tableau's settings and capabilities I realized that the format my data was in would not support the kind of implementation I wanted. That was when I returned to Pandas to do my second round of data cleaning and came out with the "all_shots.csv" dataset.

After loading that into Tableau, it became clear that my data was in the right form. Although I still needed a calculated field in order to make my X and Y columns usable in the background image, It worked much better than before, and this time I was able to easily filter my data through the different zones. I messed around for a time in Tableau, seeing what values I could get to show up in certain places and checking them each time with my data in the original ipynb file to ensure Tableau was displaying it correctly. Eventually I settled on a format for my visualization, and got to work on adding some search parameters.

Adding player search was mandatory for this project, as I wanted users to be able to search for any player who played that season and look at their individual shooting percentage for each zone. Filtered searches are very easy to create in Tableau, but they don't look very nice in their default state. I wanted the searchbox to show suggestions as a user was typing in a name. That way, if someone only knew the first name or only had a vague understanding of how to spell it (as some names could be extremely complex), they could type in what they knew, then click on the player. Tableau does not make this concept standard. Instead of just a dropdown search bar, they show up as check boxes where you have to deselect the entire dataset and then select the specific value you want to get that value to show up. Wildcard matching is a setting that can be selected to look for values that match what you put in a searchbox, but the issue with it is that it keeps the original search format of selecting boxes. Worse of, at rest the search resets to <ALL>, in which it selects the data for EVERY NBA player and prints it on the visualization, creating a huge clump of unreadable text that just defeats the whole purpose of the map to begin with. It took some messing around, but I found 

I added a second search bar that was for the teams and then connected that to the player search via the "Only Relevant Values" filter setting in Tableau. Then, a user could search for the individual team and the only players to show up in player search would be those from that team. This just made manuevering around the visualization easier. I included a filter that could take out certain shot types if users wanted to only focus on a specific zone, but it is auto-set to all as a baseline. 

Lastly, on a new worksheet I made a text table that would show a player, their field goal makes (sum), field goal attempts (sum), and field goal percentage (average) of all selected shot zones. I then applied my player and shot type filters from my first worksheet onto this one to make the table reflect the player currently selected in the search bar, and only show data relevant to the selected zones (i.e. if only mid-range shot type is selected, only the FGA, FGM, and FG% from the mid-range are shown).

All of that got put on the end-result dashboard, which was then uploaded to Tableau Public on my profile (See: [Tableau Profile](https://public.tableau.com/app/profile/owen.larimer/vizzes)).

#### Map Creation Screenshots

![image](https://github.com/user-attachments/assets/6a55f581-4da9-49a9-b5d4-957f30d877ce)
*Map creation and filter applications*

![image](https://github.com/user-attachments/assets/9a63494f-f69f-46e3-8954-f6c1ea8a5025)
*Text table creation*

### Capabilites

Now with the finished shot chart, you can view the 23-24 shooting statistics of any player who played in the 2023-24 NBA season from a multitude of zones or any singular zone, not including free throws. This kind of tool could be used for any number of future shooting data projects, or just as a way of identifying individual players' shooting skills and hot zones.

### Limitations

The main limitation of this project was the data and its collection process. Because you cannot scrape directly from the National Basketball Association's official site, you either have to find an API or website that does it for you, or manually grab the data and completely clean it yourself. I cannot be certain since I have not tried, but this type of manual collection could prove useless with certain statistics that are more categorical in nature. Any numerical value is easy to grab, clean, and convert as a data type, but when you start wanting unabbreviated team names, or notes on trading within its own column, you could run into problems if not using an API. Be mindful of any data you grab, and ensure it's cleanliness.

Another notable limitation is the fact that my search parameters within the Tableau Dashboard do not account for players traded midseason. From what I can understand from my data, the "Team" column only lists the players team at the end of the season. If a user knew a player from a specific team in which that player was traded from during that season, they would not show up in the player search box after setting the team search box to that original team. This wasn't a big deal as it does not make or break the visualization, but I can see how a user might struggle in a very specific scenario due to this fact.

### Takeaways

I learned an obscene amount from this project. Not only could I refine my data cleaning skills in Python, but it gave me a bit of a glimpse into the capabilities of excel in regards to cleaning. However, my Tableau skills were by far the most improved as a result of this project. Before this, I knew very little about the ins and outs of Tableau mapping, filters, calculated fields, and parameters. This project included all of that, and I am now much more confident in my ability to maneuver them. For example, I understand much better how to map data on a background image, how it's not difficult so much as it is choosy about the data an image accepts, and how it can be used to make data visualization so much easier to understand and so much more interesting for a user.

### From Here

The main goal of this project was to visualize scoring statistics in the NBA from last season. My initial vision was to have clickable "zones" in which a user could see important statistics from that zone, such as the scoring leader, percentage leader, worst overall, etc. In doing the project I had to realign my goals early on to fit more within my capabilities with these tools. I instead elected to show statistics for every player in every zone, which meant more data could be shown but I couldn't go as in depth into that data as I had originally imagined. At some point, there are many features I want to consider adding to this shot chart. I hope to be able to make the zones cover that entire area of the court, rather than just being a dot, and for them to light up when hovered over. I want to find a way to show statistical leaders for each shot type after clicking on them, or adding some kind of button/filter on the side that changes the shot chart to show that statistical leader. This kind of project could be massively expanded to include multiple years, with free throws and eFG% (effective field goal percentage) as additional stats. A drop down search where you could search through years could make that possible, all that is needed is the data to implement it. I think adding a player's portrait into the text table would be a great addition, and would come with its own challenges of having an image as a data type. Additionally, plenty other visualizations like this could be made regarding the various individual stats in basketball. I can see versions of this shot chart being done for assists, rebounding, win percentages for coaches and players, and locations of turnovers. There really is no constricting limit for this type of project, it's only a question of having the right data.
