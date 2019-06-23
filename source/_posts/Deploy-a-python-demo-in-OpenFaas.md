---
title: Deploy a python demo in OpenFaas
date: 2019-06-23 23:42:09
tags: OpenFaas
---

## Deploy a python demo in OpenFaas

> After a try in OpenFaas, here I make a effort to deploy my own function via OpenFaas. 

A few days ago, I play Chinese string up puzzle in QQ with a robot named xiaobing. An interesting thought came to mind that whether I could write such a function service which can help me figure the next idiom in the puzzle. So I get to write my python function.



### Collect and Pre-processing

First, I found some words in [Github repo](https://github.com/pwxcoo/chinese-xinhua/blob/master/data/idiom.json). And I make pre-process on the data to help easy get the next idoim.

##### Pre-process.py

```python
with open("idioms.json","r") as f:
  words = json.load(f)
# use dict to help find the next idoim quickly
wdict = {}
for word in words:
  l = wdict.get(word[0],[])
  wdict = l.append(word)
with open('idioms_dict.json','w') as f:
  json.dump(wdict, f, ensure_ascii=False)
```

Then you could find that the pre-processed data. You can find the idoim with its first character.

#####idioms_dict.json

````json
{
    "阿": [
        "阿鼻地狱",
        "阿党比周",
        "阿党相为"
    ],
    "哀": [
        "哀哀欲绝",
        "哀思如潮",
        "哀天叫地",
        "哀痛欲绝"
    ],
    "唉": [
        "唉声叹气"
    ],
    "挨": [
        "挨冻受饿",
        "挨风缉缝",
        "挨三顶五",
        "挨山塞海"
    ],
    "捱": [
        "捱风缉缝",
        "捱三顶四",
        "捱三顶五"
    ],
    "嗳": [
        "嗳声叹气"
    ],
    ...
}
````



### Write the python function

 To implement the Chinese string up puzzle, I write a little demo.

```python
import json
import random

def handle(req):
    with open('./function/idioms_dict.json', 'r') as f:
        words = json.load(f)
    try:
        req = str(req).strip()
        if req not in words[req[0]]:
            return 'the idiom not exist'
        next_words = words.get(req[-1], [])
        length = len(next_words)
        if length == 0:
            return "can't find the next idion"
        word = next_words[random.randint(0, length - 1)]
        return word
    except:
        return 'bad parameter'
```

##### Notice:

> Here why I use the path `./function/idioms_dict.json` is that in OpenFaas, after you deploy your function, the files in the handler folder will be placed in `./function`. So I need to get access to `./function` to get my `idioms_dict.json`.



### Deploy the python function

After working with the function, I modify the `yml` file.

##### idoim-follow.yml

```yml
version: 1.0
provider:
  name: openfaas
  gateway: http://127.0.0.1:8080
functions:
  idoim-follow:
    lang: python3
    handler: ./idoim-follow
    image: kids937/idoim-follow:latest
```

Now all work about coding have been finished yet. It's so easy, isn't it? See more detail in [my GitHub repo](https://github.com/ChaosCodes/idoim-follow-in-openfaas).

Let's start to make it deploy.

```shell
$ faas-cli build -f idoim-follow.yml &&\
  faas-cli deploy -f idoim-follow.yml
```

Wait a second to build the image, and you can see a function deployed in the OpenFaas Portal.

![deploy-idoim-follow](deploy-idoim-follow.png)



### Play with my own function

Try to check whether the function work well.

![check-idoim-follow](check-idoim-follow.png)

Amazing！Now I can get the next idoim just by typing the question idoim and invoking the function~

Moreover, I'm going to play Chinese string up puzzle with xiaobing again, and I'm determined to defeat her this time. LOL.