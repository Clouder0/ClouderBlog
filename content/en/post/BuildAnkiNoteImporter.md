---
title: "Build Anki Note Importer in Python"
date: 2021-05-04T15:14:57+08:00
slug: "build-anki-note-importer-in-python"
type: post
tags:
  - python
  - anki
categories:
  - Programming
---

## Why

Recently, we have to make many Anki cards from the specifically formatted text. The repeating work is boring somehow, thus a tool to automate it may be an option.

Some investigations have been conducted and to be honest, I've found a solution: [Obsidian to Anki](https://github.com/Pseudonium/Obsidian_to_Anki).

However, some functions of it may be a bit overhead to us. Meanwhile, our format is not compatible with their approach. I haven't looked into it further so I may be wrong. But to me, creating a tool to use is not that difficult and can best suit our requirements, so why not?

## How

The only important interface we need is to add notes to Anki, and a fantastic Anki plugin offers this ability: [Anki Connect](https://github.com/FooSoft/anki-connect), which allows us to communicate with Anki over a simple HTTP API.

For the remaining part, I'd like to create a tiny Python script to do the job for me.

Let's see the official example in Python:

```python
import json
import urllib.request

def request(action, **params):
    return {'action': action, 'params': params, 'version': 6}

def invoke(action, **params):
    requestJson = json.dumps(request(action, **params)).encode('utf-8')
    response = json.load(urllib.request.urlopen(urllib.request.Request('http://localhost:8765', requestJson)))
    if len(response) != 2:
        raise Exception('response has an unexpected number of fields')
    if 'error' not in response:
        raise Exception('response is missing required error field')
    if 'result' not in response:
        raise Exception('response is missing required result field')
    if response['error'] is not None:
        raise Exception(response['error'])
    return response['result']

invoke('createDeck', deck='test1')
result = invoke('deckNames')
print('got list of decks: {}'.format(result))
```

It's indeed simple, right?

The next thing is to figure out the method to add notes.

Let's see the sample request

```
{
    "action": "addNote",
    "version": 6,
    "params": {
        "note": {
            "deckName": "Default",
            "modelName": "Basic",
            "fields": {
                "Front": "front content",
                "Back": "back content"
            },
            "options": {
                "allowDuplicate": false,
                "duplicateScope": "deck",
                "duplicateScopeOptions": {
                    "deckName": "Default",
                    "checkChildren": false
                }
            },
            "tags": [
                "yomichan"
            ],
            "audio": [{
                "url": "https://assets.languagepod101.com/dictionary/japanese/audiomp3.php?kanji=猫&kana=ねこ",
                "filename": "yomichan_ねこ_猫.mp3",
                "skipHash": "7e2c2f954ef6051373ba916f000168dc",
                "fields": [
                    "Front"
                ]
            }],
            "video": [{
                "url": "https://cdn.videvo.net/videvo_files/video/free/2015-06/small_watermarked/Contador_Glam_preview.mp4",
                "filename": "countdown.mp4",
                "skipHash": "4117e8aab0d37534d9c8eac362388bbe",
                "fields": [
                    "Back"
                ]
            }],
            "picture": [{
                "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/c/c7/A_black_cat_named_Tilly.jpg/220px-A_black_cat_named_Tilly.jpg",
                "filename": "black_cat.jpg",
                "skipHash": "8d6e4646dfae812bf39651b59d7429ce",
                "fields": [
                    "Back"
                ]
            }]
        }
    }
}
```

This looks a bit more complicated, but we don't need much. A simplified version may clarify what we should do.

```python
{
    "action": "addNote",
    "version": 6,
    "params": {
        "note": {
            "deckName": "Export",
            "modelName": "CBasic",
            "fields": {
                "Front": "my front content",
                "Back": "my back content"
            },
            "options": {
                "allowDuplicate": true,
                "duplicateScope": "deck",
                "duplicateScopeOptions": {
                    "deckName": "Export",
                    "checkChildren": false
                }
            },
            "tags": [
                "#Export"
            ]
        }
    }
}
```

Talk is cheap, the code is below:

```python
import json
import urllib.request

def request(action, **params):
    return {'action': action, 'params': params, 'version': 6}

def invoke(action, **params):
    requestJson = json.dumps(request(action, **params)).encode('utf-8')
    response = json.load(urllib.request.urlopen(urllib.request.Request('http://localhost:8765', requestJson)))
    if len(response) != 2:
        raise Exception('response has an unexpected number of fields')
    if 'error' not in response:
        raise Exception('response is missing required error field')
    if 'result' not in response:
        raise Exception('response is missing required result field')
    if response['error'] is not None:
        raise Exception(response['error'])
    return response['result']

def addNote(front, back):
    return invoke('addNote',
        note = {
            "deckName": "Export",
            "modelName": "CBasic",
            "fields": { "Front": front, "Back": back },
            "options": {
                "allowDuplicate": True,
            "duplicateScope": "deck",
            "duplicateScopeOptions": { "deckName": "Export", "checkChildren": False }
            },
            "tags": ["#Export"]
        }
    )

result = addNote("testfront","testback")
print(result)
```

The next thing to do is implement a parser.

We have multiple kinds of formats:

- The first line as Question and the following as Answer.
- Single Line Cloze, using `**text**` for `{{c1:text}}` and etc.
- Multiple Line Cloze.
- ~~Outline Cloze~~: this kind of format is rather hard to support and I've given up.

The key is to recognize which kind of format the block is using and transform it into a note.

Let's see a test case.

```plaintext
This is a question.
This is an answer.

This is a question.
This is a multi-line answer.
Right?

This is a single line **cloze**.

This is a multiple **line** cloze.
intersting **clozes**.

In this case,
should it be **cloze** or Q&A?
I'd prefer Q&A.

$\LaTeX$ test.
$\LaTeX$ is so good.
```

The logic is clear:

- When `**` appears at the first line, it's a cloze.
- Otherwise, it's a Q&A.

Hands on.

We need to replace `\n` to `<br>` to support multiline stuff.

What' more, as `**text**` stands for bold text in markdown, I'd like to replace it with `<b>text</b>` as well.

In addition, $\LaTeX$ is frequently used, but in the format of `$\LaTeX$` which is not compatible with Anki, hence a substitution is required.

The final code:

```python
import json
import urllib.request

def request(action, **params):
    return {'action': action, 'params': params, 'version': 6}

def invoke(action, **params):
    requestJson = json.dumps(request(action, **params)).encode('utf-8')
    response = json.load(urllib.request.urlopen(urllib.request.Request('http://localhost:8765', requestJson)))
    if len(response) != 2:
        raise Exception('response has an unexpected number of fields')
    if 'error' not in response:
        raise Exception('response is missing required error field')
    if 'result' not in response:
        raise Exception('response is missing required result field')
    if response['error'] is not None:
        raise Exception(response['error'])
    return response['result']

def addQANote(front, back):
    return invoke('addNote',
        note = {
            "deckName": "Export",
            "modelName": "CBasic",
            "fields": { "Front": front, "Back": back },
            "options": {
                "allowDuplicate": True,
            "duplicateScope": "deck",
            "duplicateScopeOptions": { "deckName": "Export", "checkChildren": False }
            },
            "tags": ["#Export"]
        }
    )

def addClozeNote(text):
    return invoke('addNote',
        note = {
            "deckName": "Export",
            "modelName": "CCloze",
            "fields": { "Text": text },
            "options": {
                "allowDuplicate": True,
            "duplicateScope": "deck",
            "duplicateScopeOptions": { "deckName": "Export", "checkChildren": False }
            },
            "tags": ["#Export"]
        }
    )

def replaceBrackets(text, spliter,left,right):
    sub = text.split(spliter)
    output = ""
    for i in range(0, len(sub)):
        if(i % 2 == 1):
            output = output + left + sub[i] + right
        else: output = output + sub[i]
    return output
 

def Handle(text):
    if type(text) != str: raise Exception("A string is required!")
    text = replaceBrackets(text,'$','\\(','\\)')
    lines = text.splitlines(keepends = False)
    output = ""
    sub = text.split("**")
    if "**" in lines[0]: 
        # Cloze
        ## odd indexes are clozes
        for i in range(0,len(sub)):
            if(i % 2 == 1):
                output = output + '{{c' + str(((i + 1) // 2)) + '::' + sub[i] + '}}'
            else: output = output + sub[i]
        output = output.replace('\n','<br>')
        return addClozeNote(output)
    else:
        # Q&A
        output = replaceBrackets(text,"**","<b>","</b>")
        lines = output.splitlines(keepends = False)
        front = lines[0]
        back = ""
        for i in range(1, len(lines)):
            back = back + lines[i] + '<br>'
        print(back)
        return addQANote(front,back)

def UnitTest1():
    text = """This is a multiple **line** cloze.
intersting **clozes**."""
    print(Handle(text))

def UnitTest2():
    text = """This is a question.
This is an answer."""
    print(Handle(text))

def UnitTest3():
    text = """This is a question.
This is a multi-line answer.
Right?"""
    print(Handle(text))

def UnitTest4():
    text = "This is a single line **cloze**."
    print(Handle(text))

def UnitTest5():
    text = """In this case,
should it be **cloze** or Q&A?
I'd prefer Q&A."""
    print(Handle(text))

def UnitText6():
    text = """$\LaTeX$ test.
$\LaTeX$ is so good."""
    print(Handle(text))

UnitTest1()
UnitTest2()
UnitTest3()
UnitTest4()
UnitTest5()
UnitText6()
```

Looks good. It went through my tests and worked as expected.

## Final

A simple tool.

Requires Python3. Usage: `python3 filename.py target`

It will write a log file to the current directory.

```python
import json
import sys
import datetime
import urllib.request

def request(action, **params):
    return {'action': action, 'params': params, 'version': 6}

def invoke(action, **params):
    requestJson = json.dumps(request(action, **params)).encode('utf-8')
    response = json.load(urllib.request.urlopen(urllib.request.Request('http://localhost:8765', requestJson)))
    if len(response) != 2:
        raise Exception('response has an unexpected number of fields')
    if 'error' not in response:
        raise Exception('response is missing required error field')
    if 'result' not in response:
        raise Exception('response is missing required result field')
    if response['error'] is not None:
        raise Exception(response['error'])
    return response['result']

def addQANote(front, back):
    return invoke('addNote',
        note = {
            "deckName": "Export",
            "modelName": "CBasic",
            "fields": { "Front": front, "Back": back },
            "options": {
                "allowDuplicate": True,
            "duplicateScope": "deck",
            "duplicateScopeOptions": { "deckName": "Export", "checkChildren": False }
            },
            "tags": ["#Export"]
        }
    )

def addClozeNote(text):
    return invoke('addNote',
        note = {
            "deckName": "Export",
            "modelName": "CCloze",
            "fields": { "Text": text },
            "options": {
                "allowDuplicate": True,
            "duplicateScope": "deck",
            "duplicateScopeOptions": { "deckName": "Export", "checkChildren": False }
            },
            "tags": ["#Export"]
        }
    )

def replaceBrackets(text, spliter,left,right):
    sub = text.split(spliter)
    output = ""
    for i in range(0, len(sub)):
        if(i % 2 == 1):
            output = output + left + sub[i] + right
        else: output = output + sub[i]
    return output
 

def HandleNote(text):
    if type(text) != str: return "not string type!\n"
    if "  - " in text: return "found Outline Structure, skipping.\n"
    text = replaceBrackets(text,'$','\\(','\\)')
    lines = text.splitlines(keepends = False)
    output = ""
    sub = text.split("**")
    if "**" in lines[0]: 
        # Cloze
        ## odd indexes are clozes
        for i in range(0,len(sub)):
            if(i % 2 == 1):
                output = output + '{{c' + str(((i + 1) // 2)) + '::' + sub[i] + '}}'
            else: output = output + sub[i]
        output = output.replace('\n','<br>')
        return "Invoke successfully, return code:{}\n".format(addClozeNote(output))
    elif len(lines) >= 2:
        # Q&A
        output = replaceBrackets(text,"**","<b>","</b>")
        lines = output.splitlines(keepends = False)
        front = lines[0]
        back = ""
        for i in range(1, len(lines)):
            back = back + lines[i] + '<br>'
        return "Invoke successfully, return code:{}\n".format(addQANote(front,back))
    return "Unmathcing any format.\n"

def HandlePost(text):
    if type(text) != str: raise Exception("A string is required!")
    notes = text.split("\n\n")
    f = open("log.txt","a+")
    f.write("\n" + datetime.datetime.now().strftime("%c") + "\n")
    for note in notes:
        f.write(HandleNote(note))
    f.close()

if len(sys.argv) < 2: 
    print("Please provide a param: python.py filename.md")
    exit()
path = sys.argv[1]
print("path: " + path)
f = open(path,"r")
HandlePost(f.read())
f.close()
print("Done.")
```