## 소개
 
2013년 주간 프레시안 뷰 http://view2013.pressian.coop
 
## 준비
 
gitbook 설치

```
$ npm install gitbook -g
```

데이터 파일 구조

```
./Authors.yml
./Categories.yml
./V01
./V01/Issues.yml
./V01/S01
./V01/S01/closing.md
./V01/S01/economics.md
./V01/S01/environments.md
./V01/S01/figures
./V01/S01/figures/s01-economics-fig1.jpg
./V01/S01/northeastasia.md
./V01/S01/politics.md
./V01/S01/world.md
```

파일 및 폴더 설명

| 파일 또는 폴더 이름 | 설명 |
|---|---|
| Authors.yml | 저자 정보 |
| Categories.yml | 분류 정보 |
| Issues.yml | 호 정보(각 호별 발행일) |
| 폴더 V?? | 권 번호(Volume ??) |
| 폴더 S?? | 호 번호(Issue ??) |

저자 정보 파일(`Authors.yml`)의 예,

```yaml
- Name: 박인규
  Author-ID: pik
  Description: "관점이 있는 뉴스 <프레시안>의 초대 편집국장으로, 2003년부터 대표를 역임했다. 2013년 한국 최초로 언론 협동조합으로 전환함에 따라 이사장으로 일하고 있다. 전두환 정권에서 기자 생활을 시작한 이래, 국제 전문 기자로 활약했으며 지금도 전문성을 인정받고 있다. 지난해 미국의 안보 정책을 심도 깊게 통찰한 앤드루 바세비치의 책 <Washington Rules: America's Path To Permanent War>를 <워싱턴 룰>로 번역·출간했다."
- Name: 정태인
  Author-ID: jti
  Description: "노무현 전 대통령의 경제 가정교사로 불리며, 참여정부에서 국민경제 비서관을 역임했다. 그러나 노무현 정부가 추진한 한미FTA 체결에 반대, '반(反) FTA 전도사'로 활약했다. 당시 <프레시안>과 FTA의 문제점을 지속적으로 고발했으며, 사회적으로 큰 반향을 일으켰다. 저서로는 <착한 것이 살아남는 경제의 숨겨진 법칙>, <정태인의 협동의 경제학> 등이 있으며, <불량사회와 그 적들>과 같이 사회적 문제를 비판한 공저가 여러 권 있다."
```

분류 정보 파일(`Categories.yml`)의 예,

```yaml
- Category-ID: closing
- Category-ID: economics
- Category-ID: politics
```

호 정보 파일(`Issues.yml`)의 예,

```yaml
- Issue: 1
  PubDate: 2013-06-28
- Issue: 2
  PubDate: 2013-07-05
- Issue: 3
  PubDate: 2013-07-12
```

컨텐츠 파일(`world.md`)의 예,

```
---
Author-ID: pik
Title: "스노든 폭로가 보여준 것 : 언론의 타락과 미국의 몰락"
---
지난 23일 홍콩을 떠난 ‘세기의 고발자’ 에드워드 스노든은 28일 현재 러시아 모스크바공항의 환승구역에 머무르면서 중남미 국가인 에콰도르로의 망명을 추진 중입니다. 그의 ‘배신행위’에 분노한 미 당국이 22일 그의 여권을 취소하는 바람에 언제 그가 모스크바 공항을 떠날 수 있을지 현재로서는 불투명하다고 하는군요. 그러나 지난 9일 그의 폭로가 몰고 온 파장은 쉽게 가라앉지 않을 전망입니다.
```

템플릿 파일 및 gitbook 구조
 
빌드 스크립트에서 생성한 파일은 autogen 확장자를 추가하도록 하였음.

```
./ARTICLE_LIST.tpl.md
./CONTENT.tpl.md
./v01
./v01/cover.jpg
./v01/cover_small.jpg
./v01/economics-2013-6.autogen.md
./v01/economics.autogen.md
./v01/environments-2013-6.autogen.md
./v01/environments.autogen.md
./v01/figures
./v01/figures/s01-economics-fig1.jpg
./v01/northeastasia-2013-6.autogen.md
./v01/northeastasia.autogen.md
./v01/README.md
./v01/SUMMARY.md
```

기사 목록 템플릿 파일(`ARTICLE_LIST.tpl.md`)의 예,
 
```jinja
{% for article in articles %}
- {{ article.Author }}, {{ article.Title }} ({{ article.PubDate.year }}\. {{ article.PubDate.month }}\. {{ article.PubDate.day }})
{% endfor %}
```

컨텐츠 템플릿 파일(`CONTENT.tpl.md`)의 예,

```jinja
{% for article in articles %}
- {{ article.Author }}, {{ article.Title }} ({{ article.PubDate.year }}\. {{ article.PubDate.month }}\. {{ article.PubDate.day }})
{% endfor %}
```

목차 파일(`SUMMARY.md`)의 예,
 
```markdown
# Summary
* [펴내며](README.md)
* [세계](world.autogen.md)
   * [2013년 6월](world-2013-6.autogen.md)
* [경제진단](economics.autogen.md)
   * [2013년 6월](economics-2013-6.autogen.md)
```

빌드 스크립트

gitbook 용 파일 생성 코드(`generator.py`)

```python
import os
import sys
import glob
import shutil
import re
import yaml
from jinja2 import Template
GITBOOK_PREFIX = 'gitbook'

def find(ls, k1, v1, k2):
    v2 = None
    for dic in ls:
        if dic[k1] == v1:
            v2 = dic[k2]
    return v2

def load(volume_num=1):
    # Load global configurations
    f = open('data/Authors.yml', 'r')
    authors = yaml.load(f.read())
    f = open('data/Categories.yml', 'r')
    categories = yaml.load(f.read())
    f = open('data/V{0:02d}/Issues.yml'.format(volume_num), 'r')
    issues = yaml.load(f.read()) # 각 볼륨별로 한 권이 책이 되도록 구성
    contents = list()
    for issue in issues:
        # Isssues.yml 파일에 설정되어 있는 데이터만 로드
        issue_num = issue['Issue']
        pubdate = issue['PubDate']
        fnames = glob.glob("data/V{0:02d}/S{1:02d}/*.md".format(volume_num, issue_num))
        content = dict()
        for fname in fnames:
            category_id = os.path.basename(fname)[:-3]
            f = open(fname, 'r')
            # 첫 document 혹은 두번째 document를 점검하고 나머지는 전부 body로 인식
            documents = re.split('^---$', f.read(), flags=re.MULTILINE)
            header = documents.pop(0)
            if header == '':
                header = documents.pop(0)
            header = yaml.load(header)
            body = '\n'.join(documents)
            author_id = header['Author-ID']
            content['Volume'] = volume_num
            content['Issue'] = issue_num
            content['PubDate'] = pubdate
            content['Category-ID'] = category_id
            #content['Category'] = find(categories, 'Category-ID', category_id, 'Title')
            content['Author-ID'] = author_id
            content['Author'] = find(authors, 'Author-ID', author_id, 'Name')
            content['Title'] = header['Title']
            content['Body'] = body.strip()
            contents.append(content.copy())
    return contents

def write(articles):
    category_id = articles[0]['Category-ID']
    pubdate = articles[0]['PubDate']
    volume = articles[0]['Volume']
    volume_dir = GITBOOK_PREFIX+'/v{0:02d}/'.format(volume)
    template = Template(open(GITBOOK_PREFIX+'/CONTENT.tpl.md', 'r').read())
    fname = volume_dir+'{0}-{1}-{2}.autogen.md'.format(category_id, pubdate.year, pubdate.month)
    f = open(fname, 'w')
    f.write(template.render(articles=articles))
    for article in articles:        issue = article['Issue']
        fnames = glob.glob('data/V{0:02d}/S{1:02d}/figures/*'.format(volume, issue))
        for fname in fnames:
            figures_dir = volume_dir+'figures/'
            if not os.path.exists(figures_dir):
                os.mkdir(figures_dir)
            shutil.copy(fname, figures_dir+os.path.basename(fname))

def write_category_front(category_id, category_contents):
    volume = category_contents[0]['Volume']
    volume_dir = GITBOOK_PREFIX+'/v{0:02d}/'.format(volume)
    template = Template(open(GITBOOK_PREFIX+'/CATEGORY.tpl.md', 'r').read())
    fname = volume_dir+'{0}.autogen.md'.format(category_id)
    with open(fname, 'w') as f:
        f.write(template.render(articles=category_contents))

def render(contents):
    # Find unique category-ids
    category_ids = set([content['Category-ID'] for content in contents])
    categories = { category_id: list() for category_id in category_ids }
    # 같은 카테고리별로 정렬
    for content in contents:
        categories[content['Category-ID']].append(content)
    # 년-월 순서로 정렬
    for category_id in category_ids:
        categories[category_id] = sorted(categories[category_id], key=lambda x: x['PubDate'])
    for category_id, category_contents in categories.items():
        write_category_front(category_id, category_contents)
    for category_id, category_contents in categories.items():
        # pubdate 와 articles 초기화
        pubdate = category_contents[0]['PubDate']
        articles = [ category_contents[0] ]
        # 날짜 순서로 정렬된 상태라 가정, category_contents 리스트는 2개 이상이라 가정
        for content in category_contents[1:]:
            if pubdate.month == content['PubDate'].month:
                articles.append(content)
            else:
                # 모아놓은 articles 출력
                write(articles)
                # 새로운 articles 리스트 시작
                articles = [ content ]
            pubdate = content['PubDate']
        # 마지막으로 생성된 articles 리스트 출력
        write(articles)

def main():
    contents = load()
    render(contents)

if __name__ == '__main__':
    main()
```

빌드 스크립트(`build.sh`)

```bash
#!/bin/bash
python generator.py
gitbook build -o _site/ ./gitbook/v01/
gitbook epub -o _ebook/weekly.epub ./gitbook/v01/
gitbook mobi -o _ebook/weekly.mobi ./gitbook/v01/
gitbook pdf -o _ebook/weekly.pdf ./gitbook/v01/
```

문제점

앞서 방법으로 생성한 gitbook 파일을 기반으로 새로 저장소를 추가한 후 gitbook 으로 push 함.

```
$ git remote add gitbook https://push.gitbook.io/pressian/weekly-pressian-view.git
$ git push -u gitbook master
```