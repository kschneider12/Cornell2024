Table of Contents:
    Introduction
    Functions:
        cluster_using_all_wks;
        coord_clusters2;
        mapping;
        using_all_weeks;
        graph_comparison;
        daily_no_avg;
        show_site_data;
        graphing (evap);
        get_all_data (evap);
        process_eddi3 (evap);
        find_similarities;
        within_range;
        get_station_info;
        get_dates;

Introduction:
Over the past few months, I have been working with multiple datasets in order to help define flash droughts. Initially starting with groundwater
data, I eventually focused on streamflow data, precipitation data, evaporation data, and eventually soil moisture data. As groundwater is the most
sporadic and scarce dataset (and therefore the least useful), I will not be addressing it in this writeup. Since this data covers daily coverage
between the 1950s to present day, I worked with hundreds of thousands of numbers to extract just a few important occurrences that may exhibit
signs of flash drought. In this folder lies a powerpoint with different graphs and comparisons across datasets exhibiting these occurrences.
To enable further research with my work, I am leaving detailed descriptions of how these functions I utilized work, and how an outside party can
run their own analysis with it.

Functions:

cluster_using_all_wks:

    This function is the ultimate control panel for most of the other functions I worked on with streamflow. It generates clusters of stations
    across a given state, pulls out desired clusters, and analyzes them, comparing all of their changes in streamflow data across a given period of
    time, and sorting them. It then can graph the occurrences of a given percentile, showing how many occurrences of that percentile occurred in each
    month of the year, and can show number the occurrences of the 1% of both streamflow and rainfall over all years simultaneously. Additionally, when
    percentile = 10%, the 1%, 2%, 5%, and 10% lowest value is listed, showing where each percentile starts. Finally, it can print every date/value from
    a percentile.
    parameters:     (state, cluster, cluster2, start_date, end_date, percentile, scope, typ, scopetyp, window)
    state - identifies which state to pull the data from. This works for station based data, such as streamflow or groundwater.
        type: string, format: "NY" or "MA"
    cluster - for some states, multiple clusters want to be identified in different regions. For NY, I used 4 different regions. cluster
    identifies the number of clusters that should be created in each region. (ranking from the best clusters down to the worst)
        type: int
    cluster2 - identifies which cluster to look at. for states with multiple regions for clusters, this number starts at 0 being the first cluster in
    the first region. if the variable cluster = 5, cluster2 = 6 would give the first cluster in the second region of that state.
        type: int
    start_date - a string of the start year for data collection and analysis
        type: string, format: "1950"
    end_date - a string format of the end year for data collection and analysis
        type: string, format: "2023"
    percentile - since this function generates graphs of different percentiles, this variable is what percentile is shown and printed
        type: int
    scope - identifies how many days the data sample will be taken from around a particular date. Example: scope = 2, averaged after 7/16/XXXX would
    average 7/16/XXXX-7/18/XXXX
        type: int
    typ - identifies how the data will be collected. Two options, averaging the data or taking the minimum from the data are available. Both take
    the sample created by scope and typ determines which single value will be used later in the function
        type: string, format: "min" or "avg" ONLY
    scopetyp - used in earlier versions of analysis, no longer necessary but can be utilized. determines where the data collection is centered around.
    "front" (the only option currently) takes the data starting at the date, and counts forwards scope amount. Another feature, perhaps "middle" could
    be implemented to change the average or minimum from data collection to be centered around the date instead.
        type: string, format: "start" ONLY
    window - determines how many weeks the comparison occurs. with scope = 7, typ = front, and window = 1, one week would directly be compared to the
    following week. The window begins counting from the initial date, and is unaffected by the scope variable. decimals for window do work to an extent
    (I believe)
        type: int

coord_clusters2:

    This function is the subfunction of cluster_using_all_wks that generates the clusters themselves, and returns a list of clusters, organized in
    the format: [[calculated cluster strength for sorting, stationIndex1, stationIndex2, stationIndex3, stationIndex4, stationID1, 
    stationID2, stationID3, stationID4], [repeat,,,,,,,,], [repeat,,,,,,,,,]]
    This returned list is every calculated cluster, meaning if cluster2 is 4, and there are 4 regions, there will be 16 returned clusters.
    This is also why cluster2 starts at the start of all the clusters, because all it does is look through this output to find the selected cluster.
    above coord_clusters2 are constants, which contain station codes for good stations (meaning they have enough data) This enables the clusters
    to only be generated from good stations. This was calculated through get_station_info, which is covered later.
    at the top of the function, num_of_stations determines how many additional stations can cluster with the first station. It only works at 3, with
    4 stations total making up a cluster. Some minor tweaking could change this though. EXPONENT, MULTIPLIER, and DIVIS are all used to generate the
    value that is used to compare the stations to each other. The only important information about that needed is that the calculation is based on
    proximity of the stations and the amount of data at each one. I found a good fit with the current values, but messing with them would produce
    different results.
    IMPORTANT!!!!!-----------
    At the bottom of coord_clusters2 is a section that creates the segments that a state is cut into. This is currently coded for NY, as nearly all
    my analysis took place there. If you want to look at clusters in other states, it should still work fine, but the cluster variable may be a little
    sporadic. However, minimal trial and error would enable you to highlight the cluster you're looking for. If you wish to create your own regions
    of a different state, all you need to do is change the -76, -78, 43, etc, as those are coordinates, to whichever coordinates you want to limit
    your regions to.
    parameters:     (state,num_clusters, data)
        state - see cluster_using_all_wks
        num_clusters - see cluster_using_all_wks (cluster)
        data - imported data from cluster_using_all_wks, dictionary of dates and values, generated in show_site_data

mapping:
    
    This function generates clusters in the same way as generated clusters, as it calls coord_clusters2 inside it, but it just exports a map of the given
    state, with all the stations in GOOD_STATIONS from that state, with the selected cluster highlighted. It is solely a visual aid for finding specific
    clusters and for presentation purposes.
        parameters:         (state, num_clusters, highlighted)
            state - see cluster_using_all_wks
            num_clusters - see cluster_using_all_wks (cluster)
            highlighted - the 4 values of the station codes of the desired cluster, found from coord_clusters2. Example listed at top of 
            cluster_using_all_wks (It is nested in an if statement so with large analysis, it doesn't print every time, just once per region)

using_all_weeks:

    given a particular station, this is the grandfather function for calculating and sorting all of the changes over time, over all time.
    Starting by importing the data for that station, this function then sorts all the data into groups of weeks (for faster analysis) before 
    iterating through weeks of the year and calling graph_comparison (covered later) This function returns the percentile of all the desired data.
    If percentile = 1%, the top 1% drops in the data will be returned in a [[value, date range], [....], [....] format]
    However, the last value is also returned, which contains the standard deviation and mean of all the data. This is used to normalize the data across
    different stations later.
        parameters:     (data, date, start_date, end_date, index_of_site, percent, scope, typ, scopetyp, window)
            data - see coord_clusters2
            date - the date that analysis begins (should stay 01-01)
                type: string, format: "01-01"
            start_date - see cluster_using_all_wks
            end_date - see cluster_using_all_wks
            index_of_site - if data import contains multiple sites, this is the index of which site to look at. When used in cluster_using_all_wks, it
            is always 0, because data only consists of one station at a time.
                type: int
            percent - see cluster_using_all_wks (percentile)
            scope - see cluster_using_all_wks
            typ - see cluster_using_all_wks
            scopetyp - see cluster_using_all_wks
            window - see cluster_using_all_wks

graph_comparison:

    This function is what takes the start date, and compares the average of those days to the second range of dates. It takes two date segments and
    compares the two segments to each other across all years. For example, if fed "01-01" and a 2 week window for comparison, it would compare the first
    week of every year to the third week of that same year. It returns a list of values, such as the median, mean, the changes themselves, and the standard
    deviation. This function is repeated 52 times to look at every week across all years in the dataset.
    parameters:     (graph, date, start_date, end_date, window, plotting, percentile, scope, typ, scopetyp)
        graph - A list of two dictionaries, one for the first segment for analysis, and one for the other segment of analysis. It is created
        in using_all_weeks, and was implemented separate from the data (seen in using_all_weeks) as optimization to minimize search time. To use
        graph_comparison correctly, all you need is the first dictionary in the list to have all the dates you care about for the first section, and the
        same for the second section of dates you are comparing the first one to.
            type: [{},{}] list of dictionaries (2 only)
        date - see using_all_weeks
        start_date - see cluster_using_all_wks
        end_date - see cluster_using_all_wks
        window - see cluster_using_all_wks
        plotting - boolean that graphs the results instead of returning a value. only used in the first few weeks of analysis.
            type: boolean
        percentile - different from percentile in cluster_using_all_wks, this used to return a percentile of high values that occured that day over
        different years.
            type: int
        scope - see cluster_using_all_wks
        typ - see cluster_using_all_wks
        scopetyp - see cluster_using_all_wks

daily_no_avg:

    This function takes a datatime object and a dictionary of values, and iterates through scope amount, averaging or returning the minimum of
    the data in that set. It is used up a level by graph_comparison, as these two returned values are what are compared to create the change over time.
    The title is misleading, as it is daily_no_avg, but its function solely is to collect multiple datapoints to return only one that can summarize the data.
    parameters:     (graph, start, end_date, start2, scope, typ, scopetyp):

                    (graph[0], start, end_date, start, scope, typ, scopetyp)

        graph - just one of the two dictionaries fed into graph_comparison, so it is a {date: value} dictionary.
            type: dictionary
        start - a datetime object of the date analysis begins
            type: datetime
        end_date - see cluster_using_all_wks
        start2 - datetime object of where the analysis begins, solely to help attach the correct year to the calculated difference
            type: datetime
        typ - see cluster_using_all_wks
        scopetyp - see cluster_using_all_wks
    
show_site_data:

    This function solely takes in the raw data, and creates a dictionary of all the values inputted, specifically for streamflow. It returns
    a dictionary in the format {YYYY/MM/DD: Value}
    parameters:     (data, site, print_data)
        data - the raw data from the file
            type: string
        site - an integer that represents the index in the data file that holds all the values for a single specified site.
        This integer can be found through get_site_info, or is 0 if only one stations' data is imported.
            type: int
        print_data - a boolean whether or not to list the data as it is formatted properly
            type: boolean

graphing:

    Used in the evaporation.ipynb file, this is used to graph evaporation, soil moisture, or any other data set in the proper format. It
    counts how many occurances happen each month and graph months and occurances on a bar graph. The dataset is the percentile given by the user.
    This enables the user to see which months have the greatest drops in month XX compared to month XX.
    parameters:     (data, end_date, percentile)
        data - a list of strings, all in the printable format ['Change in Value, YYYY/MM/DD - YYYY/MM/DD', '', '']
            type: list
        end_date - see cluster_using_all_wks
        percentile - see cluster_using_all_wks

get_all_data:

    Used in the evaporation.ipynb file, this collects all of the data for evaporation and exports it to a file called all_data_saved.json. 
    The region identified for where collection is specified in process_eddi3. It iterates through every day of the year (or a smaller section such as
    April to October and collects all of that data). The line -for i in range(91,274):- identifies which days to collect. The first number represents which
    day to start collection (here April 1) and the last number identifies which day of the year to end collection. Leap days are skipped.
    parameters: (end_date)
        end_date - see cluster_using_all_wks

process_eddi3:

    Used to gather evaporation data and written by Keith Eggleston, this function was only modified to return the base values, without 
    doing any analsis on it. This function returns a dictionary of dates and values, in the format {YYYY/MM/DD: Value}
    parameters:         (ryear, rmonth, rday)
        ryear - start year for analysis
            type: int
        rmonth - month of analysis
            type: int
        rday - day of analysis
            type: int

find_similarities:

    This function searches through every week of the year, and combines all values in the parameter variables that fall within a month of that week. This 
    enables the user to see where there are overlaps (presented in the powerpoint) across all time, and what those values and dates were for those overlaps.
    It does not return anything, but instead just prints the results to the console.
    parameters:         (sf,ev,pc,sm)
        sf - Streamflow. A list of lists of strings, with the first list containing strings in the same format as data (see graphing) represting a 1 week
        difference. The second list is 2 weeks apart, the third list is 3 weeks apart, and the 4th list is 4 weeks apart. The week differences can be whichever
        you want, as long as they are the same across sf, ev, pc, and sm. This list of 4 lists is not created through code, although this could be implemented
        gracefully. I calculated the 4 lists for the week differences and then manually combined it into one list, saved as a constant for faster analysis.
            type: [['',''],['',''],['',''],['','']]
        ev, pc, sm - same a streamflow but are for Evaporation, Precipitation, and Soil Moisture. You can add more values and modify the code if desired.

within_range:

    This function compares a day in time to an event from the parameters in find_similarities. For example, it takes a week in May in 1986 and an event in June
    of 1986 and compares if the dates are close enough (currently a month apart), and if so, returns True.
    parameters:         (a, b)
        a - a datetime object representing a day that the event is compared to.
            type: datetime
        b - one string pulled from the parameters in find_similarities
            type: string

get_station_info:
    This function counts all the values recorded for a station (for streamflow or groundwater) and reports how many values it has recorded.
    In groundwater, it can declare how frequent the data is recorded. However for streamflow, most of the data is always daily. It can report the 
    first year of data recording, as well as the coordinates and station code.

    parameters:         (data, start_date, end_date)
        data - the dictionary-formatted data found through show_site_data
        start_date - see cluster_using_all_wks
        end_date - see cluster_using_all_wks

get_dates:
    this function returns a list of all the dates at the beginning of the week across a year. This is used in various functions to help sort or 
    organize data. Not useful indepentently.
    parameters:         (date, start_date, end_date)
        date - the start date for recording, in "01-01" format
            type: string
        start_date - see cluster_using_all_wks
        end_date - see cluster_using_all_wks
