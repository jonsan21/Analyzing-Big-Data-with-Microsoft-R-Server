# The NYC Taxi data

To see how we can use MRS to process and analyze a large dataset, we use the [NYC Taxi dataset](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml). The raw datasets span over multiple years and consists of a set of 12 CSV files for each month of the year.  Each record (row) in the file shows a Taxi trip in New York City, with the following important attributes (columns) recorded: 
  - the date and time the passenger(s) was picked up and dropped off, 
  - the number of passengers per trip, 
  - the distance covered, 
  - the latitude and longitude at which passengers were picked up and dropped off, 
  - payment information such as the type of payment and the cost of the trip broken up by the fare amount, the amount passengers tipped, and any other surcharges.

Each raw CSV file is about 2 Gbs in size, so 6 months worth of it amounts to 12 Gbs. That's usually more than available memory on a single personal computer.  A server can have much larger memory capacity, but if a server is used by many users at once, R can very quickly run out of memory.