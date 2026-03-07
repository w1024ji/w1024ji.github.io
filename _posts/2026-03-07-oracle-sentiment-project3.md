오늘도 어김없이 문제가 있었다. 애매한 문제이다.
AWS에 생활비를 쪼개 쓰고 있어서 glue job 으로 하는 와중에, 1월 데이터를 위해 10개의 warz 를 열었다.
그리고 약 5MB 를 S3 버킷의 silver 폴더에 올리는 걸 성공했다. Spark를 사용했다. 약 20분이 걸렸다.

그리고 약 1달러가 청구되었다.
후달달..

하지만 문제는 이제부터이다. silver -> gold 를 함과 동시에 TextBlob 라이브러리로 텍스트의 감정 평가를 했는데, 아뿔싸. 
필터링을 성공적으로 통과한 기사가 고작 10개밖에 되지 않는다는 거다.



<img width="827" height="168" alt="image" src="https://github.com/user-attachments/assets/183efc01-b1b7-4fc8-afe9-f10cb146a11e" />



CommonCrawl 에서 더 많은 warz 를 다시 건드리기에는 가격이 부담스러웠다.

일단 2월과 3월까지 해보고 다시 1월로 돌아와서 그때 모수를 늘리든가, 아니면 크롤링을 commoncrawl 이 아니라 유명한 사이트 몇 개로 한정하고 돌리는 방법도 생각할 예정이다.
대신, 2월의 common crawl 은 20개의 파일로 2배 늘렸다. 그리고 영어가 아닌 다른 나라 언어를 최대한 배제하려고 노력했다.

20개의 파일로 늘려서 그런지, 시간도 거의 2배인 약 40분이 걸렸다.
(참고로, glue job 은 타임아웃을 잘 정해야 한다. 안 그러면 리소스만 쓰고 돈만 더 내야 하니까 말이다.)

<img width="1482" height="276" alt="image" src="https://github.com/user-attachments/assets/099f0a4e-aebc-4a84-bcc5-03ff5825e980" />


그 결과, S3에는 1월의 두 배인 11MB 가 2월 폴더에 생성되었다.
로컬에서 같은 코드를 돌린 결과, 아래 이미지와 같다.



<img width="784" height="143" alt="image" src="https://github.com/user-attachments/assets/ce9b1833-4195-4fba-b924-5eb7852ca8fc" />



148개의 기사에서 감정 텍스트를 읽은 결과, 0.0809 로 1월보다 0에 가까워졌다.

-----


다음에는 3월 크롤링을 - 이건 commoncrawl 이 아닌 몇 개의 저명한 뉴스 사이트만을 이용할 예정이다 - 람다로 매일 몇 개씩 모을 수 있도록 만들어볼 생각이다.
성공할지.. 걱정이다.
