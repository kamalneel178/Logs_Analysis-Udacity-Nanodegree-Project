## Logs Analysis Project - Udacity Full Stack Web Developer Nanodegree
An internal reporting tool that uses information of large database of a web server and draw business conclusions from that information.
(Project from [Full Stack Web Development Nanodegree](https://in.udacity.com/course/full-stack-web-developer-nanodegree--nd004/))

#### The project drives following conclusions:
* Most popular three articles of all time.
* Most popular article authors of all time.
* Days on which more than 1% of requests lead to errors.

#### DESCRIPTION
For this project, my task was to create a reporting tool that prints out reports( in plain text) based on the data in the given database. This reporting tool is a Python program using the `psycopg2` module to connect to the database. This project sets up a mock PostgreSQL database for a fictional news website. The provided Python script uses the psycopg2 library to query the database and produce a report that answers the following three questions:

1. What are the most popular three articles of all time?
2. Who are the most popular article authors of all time?
3. On which days did more than 1% of requests lead to errors?

#### RUNNING THE PROGRAM
1. To get started, I recommend the user use a virtual machine to ensure they are using the same environment that this project was developed on, running on your computer. You can download [Vagrant](https://www.vagrantup.com/) and [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds_5_1) to install and manage your virtual machine.
Use `vagrant up` to bring the virtual machine online and `vagrant ssh` to login.

2. Download the data provided by Udacity [here](https://d17h27t6h515a5.cloudfront.net/topher/2016/August/57b5f748_newsdata/newsdata.zip). Unzip the file in order to extract newsdata.sql. This file should be inside the Vagrant folder. 

3. Load the database using `psql -d news -f newsdata.sql`. 

4. Connect to the database using `psql -d news`.

5. Create the Views given below. Then exit `psql`.

6. Now execute the Python file - `python logs.py`.


#### Installing the dependencies and setting up the files:

1. Install [Vagrant](https://www.vagrantup.com/)
2. Install [VirtualBox](https://www.virtualbox.org/)
3. Download the vagrant setup files from [Udacity's Github](https://github.com/udacity/fullstack-nanodegree-vm)


#### Start the Virtual Machine:

1. Open Terminal and navigate to the project folders we setup above.
2. cd into the vagrant directory
3. Run ``` vagrant up ``` to build the VM for the first time.
4. Once it is built, run ``` vagrant ssh ``` to connect.
5. cd into the correct project directory: ``` cd /vagrant/log_analysis ```


## Expected Output: 
    Calculating Results...
    TOP THREE ARTICLES BY PAGE VIEWS:
        (1) "Candidate is jerk, alleges rival" with 338647 views
        (2) "Bears love berries, alleges bear" with 253801 views
        (3) "Bad things gone, say good people" with 170098 views
    TOP THREE AUTHORS BY VIEWS:
        (1) Ursula La Multa with 507594 views
        (2) Rudolf von Treppenwitz with 423457 views
        (3) Anonymous Contributor with 170098 views
    DAYS WITH MORE THAN 1% ERRORS:
        July 17, 2016 -- 2.263% errors


### Views :
* <h4>popular_articles</h4>
```sql
create or replace view popular_articles as
select title, count(title) as views from articles,log
where log.path = concat('/article/',articles.slug)
group by title order by views desc
```
* <h4>popular_authors</h4>
```sql
create or replace view popular_authors as
select authors.name, count(articles.author) as views from articles, log, authors
where log.path = concat('/article/',articles.slug) and articles.author = authors.id
group by authors.name order by views desc
```
* <h4>log_status</h4>
```sql
create or replace view log_status as
select Date,Total,Error, (Error::float*100)/Total::float as Percent from
(select time::timestamp::date as Date, count(status) as Total,
sum(case when status = '404 NOT FOUND' then 1 else 0 end) as Error from log
group by time::timestamp::date) as result
where (Error::float*100)/Total::float > 1.0 order by Percent desc;


