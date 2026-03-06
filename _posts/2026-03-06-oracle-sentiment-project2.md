It has been almost a week since I started this project.
By the way, I am doing a graduation project in this semester (Capstone Project) with my teammates, and it looks more complicated than I expected.
I hope I can have my time to do this oracle project while doing another (and big) project ... 

----

I just created another S3 bucket and created three folders - bronze, silver and gold - to more specify the data from commoncrawler. 
CommonCrawler is huge, huge data in aws, so.. I have to work with Spark. Which takes lots of resources, so DON'T FORGET TO SPECIFY FEW WORKERS! 
'Cause I did with default setting in Glue, and the next day I saw an unexpected cost in my console. The reason was the amounts of workers I used in Glue.

Once I run Glue Job, it takes about 10 mintues. 
The 'Job Details' are: G 1X, 2 workers, 15 min of job timeout.
Also, I added libraries: warcio, beautifulsoup4.

When the job is stuck with error, I go straight to CloudWatch - log management - to see what had happended. Reading log mostly helps with fixing errors.
