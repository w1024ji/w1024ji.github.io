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


----


오늘은 저번에 작성한 Glue Job 을 더 수정하고 s3 의 silver 폴더의 1.5 M 정도 되는 json 파일들을 생성했다.
특히 footer 를 제외하니까 용량이 좀 더 가벼워지고 정돈되었다. footer 에 오라클 관련 단어가 포함된 단어가 있으니까 자꾸 silver -> gold 로 가는 파이썬 함수를 실행하면 관련 없는 기사들이 결과 csv 에 포함되길래 Glue Job 에서 footer 를 삭제하도록 했다. Silver 계층에 있는 데이터들을 '깨끗한 공통 텍스트' 로 만들고 싶었다.

그 다음에 로컬 - vscode - 에서 silver -> gold 로 필터링해서 최종 csv 를 만드는 작업을 했는데 최종 오라클 뉴스가 4개를 얻었다.
적다고 생각하지만 일단 다음에 손을 댈 때는 이대로 진행할 생각이다. 끝까지 파이프라인을 성공 시키고 난 다음에 (Lambda 까지) 데이터의 양을 늘리거나 필터링을 수정해야 겠다.

일단 오늘은 여기까지 해야겠다.
졸프를 해야 한다..

---

참고로 이 프로젝트는 내 리포지토리의 oracle-sentiment-project 에 있다. Glue Job은 2버전 파일에 있다. 1 버전은 삭제할 예정이다. (실패한 거라)


