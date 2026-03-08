I started working with AWS Lambda for the articles on March.
This time, I am not using Common Crawl - because it is for past news -, but use Google RSS news.
Crawled past news of past 7 days, and tried to get the sentiment out of each article using TextBlob.

It wasn't my first time using Lambda, but I have not used it for crawling and parsing, so obviously I ran into some errors.

<img width="1806" height="720" alt="스크린샷 2026-03-08 135520" src="https://github.com/user-attachments/assets/d4ddebbf-e913-4269-8b76-faa9b95a2fe9" />


---

Errors encountered and solved today:
- Well, Python 3.14 is not suitable for the libraries I am using for this project! So I had to roll back to 3.12 version. Which is compatible.
- The function needed Layer, so I added it with creating zip file containing python libraries needed. (with Linux version)
- It didn't worked well at first because my code got blocked from the sites, so I had to modify some of my codes to not be blocked.

---

After looking at the textblob results of Jan, Feb and first week of March, I found out they are getting close to 0.
Jan (0.11) -> Feb (0.08) -> first week of March (0.00x..)
Perhaps because Oracle is just before announcing their earning release?
Lots of things are going on with Oracle. Inside and outside.

Is AI bubbled? There are many books and articles, peoples talk about this hot issue.
Hmm.. In my opinion, AI is high likely to become a common sense in a few years, but I can't guess how much.
Of course, there are layoffs - which I also concern (bc what should I do? I am really struggling with my future job) -, environment problem with AI (usage of water and land), datacenter .. so on


