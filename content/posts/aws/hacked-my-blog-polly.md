---
layout: post
title: "I hacked my blog to let AWS Polly create podcast over it"
date: 2020-11-26
polly: https://madeddu.xyz/mp3/aws/hacked-my-blog-polly.mp3
categories:
- aws
- miscellaneous
- life
tags:
- polly
- aws
- serverless
- blog
---

### Prelude

Hi guys, after a series of back-to-the-future-I-didn't-have-time-to-write-new-things... I'm back. What happened in the last months... ok, covid19 put the whole world in trouble, I bought an apartment, I opened a company and I resign my contract. Really. Nothing. Special. But TODAY - I wanna talk about a project I have since a while, and I worked on a boring Sunday afternoon: I hacked my blog to let Polly read it for you! 😎 😎 😎

### What is Polly?

For those of you who never heard the word *Polly* before, here it is:

> AWS Polly is a service that turns text into lifelike speech, allowing you to create applications that talk and build entirely new categories of speech-enabled products.

I already made some experiments in the past (look at [A serverless OCR with Polly and Rekognition unveils the power of stack inheritance in CDK](https://madeddu.xyz/posts/aws/cdk/serverless-ocr/)) and I've always been fascinated by this:

<div class="img_container"><img src="https://i.imgur.com/CRPWma7.png" style="width: 100%; marker-top: -10px;"/></div>

AWS uses a lot of its own services to build cool things for their own purpose. One cool thing they do is to provide a spoken version of their blog posts and this is done through AWS Polly.

<div class="img_container"><img src="https://i.imgur.com/seOc82n.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Step by step

The first thing was to actually extract the text from the blog. I decided to do it the hard (better, the stupid) - way - that is... using my old friends `BeautifulSoup` Python parser library. The reason? Parsing markdown was boring and more difficult than parse the rich HTML output (I use [Hugo](https://gohugo.io/) to build my website). So I decided to remove the huge amount of dust from my old Python script to parse things and create my mp3 Polly files.

#### Step 1 of 6: __get_blog_post_urls()

The first thing to do to let AWS Polly Read your articles - and doing it using HTML - is to get your post URLs. You can certainly get your content from a local build, but I did it by parsing my blog.

My blog is hosted under a [CloudFront CDN](https://aws.amazon.com/cloudfront/?nc1=h_ls) and served under `https://madeddu.xyz` domain name. Moreover, my articles are served under the `/posts` path as many other blogs built using Hugo (I guess). A simple script like this one can get all my article URLs:

{{< highlight python >}}
def __get_blog_post_urls():

    # create request
    r = requests.get(BASE_URL)
    index = 2

    # accumulate blog post urls
    urls = []

    # go ahead with pages until
    while True:

        # parse page
        soup = BeautifulSoup(r.text, features="html.parser")

        # find all href
        for a in soup.findAll('a', href=True):

            # get only post
            if a['href'] != f"{BASE_URL}/posts/" and f"{BASE_URL}/posts/" in a['href'] and "/page/" not in a['href']:
                
                # append urls
                urls.append(a['href'])

        # create request
        try:
            r = requests.get(f"{url}/posts/page/{index}")
        except:
            return urls
        index += 1

    # return result
    return urls
{{< / highlight >}}

I just loop through pages, until I get a `200` response (starting from page 1 with no number and then going ahead with 2, 3, etc) and then I collect all links to my blog excluding my landing links and the paginator links. The results are pretty simple to imagine: it's a long list like the following one.

{{< highlight txt >}}
https://madeddu.xyz/posts/aws/cdk/serverless-ocr/
https://madeddu.xyz/posts/aws/cdk/producer-consumer/
https://madeddu.xyz/posts/aws/cdk/uploader-stack/
https://madeddu.xyz/posts/aws/cdk/contact-form/
https://madeddu.xyz/posts/life/life-as-a-software-engineer/
https://madeddu.xyz/posts/aws/cdk/cloudformation-to-cdk/
...
and so on
{{< / highlight >}}

I could use directly my filename because I use the permalink, but I also leverage folder structure to define my categories so I decided to parse my blog entirely. It can be done in a better and smarter way but my intention was to follow [GTD](https://en.wikipedia.org/wiki/Getting_Things_Done): if you are already bored at death I shared in a Github Gist the actionable tasks I wrote, entirely, in a single python script.

#### Step 2 of 6: __get_page_content()

With the first step, we collected the links to the articles. The second step is to get the content of the theme. This is not-as-simple as you can imagine, because...even in my super light Hugo template, there's A LOT of unused outer text - and I regret not having chosen a pure-text theme for my blog. So after a bit of Hugo Theme analysis, I wrote the simplest script I could think of:

{{< highlight python >}}
def __get_page_content(url, number_of_words=NUMBER_OF_WORDS, final_sentence=FINAL_SENTENCE):

    # create request
    r = requests.get(url)

    # parse page
    soup = BeautifulSoup(r.text, features="html.parser")

    # get all paragraph
    paragraphs = soup.find("div", {"id": "main"}).findAll("p")

    # accumulate outer text
    page_with_no_code = ""

    # for each found paragraph
    for paragraph in paragraphs:

        # exclude portion of code
        if paragraph.findAll("div", {"class": "highlight"}):
            continue

        # get the text out of the paragraph
        text = paragraph.text.strip()

        # exclude a common header
        if "Subscribe to my newsletter to be informed about my new blog posts, talks and activities." in text:
            continue

        # accumulate page text
        page_with_no_code += text+" "

    # return result
    return f"{page_with_no_code[:number_of_words]}{FINAL_SENTENCE}"
{{< / highlight >}}

The method `__get_page_content()` just looks for every `p` tags inside my main `div` and then, in order, excludes:

- the `highlight` div that contains my code snippets;
- the `Subscribe to my newsletter...` paragraph - the simplest thing was to actually look for the text, is not the best solution but it worked pretty well :)

The method then cut the content to a fixed number of words: I didn't want to produce too-much-long audio for a single post, but just the first portion of it, like a few paragraphs. This part can be enhanced for sure because I just cat a long string, without considering words or sentences but it works for my interest - that is, provide you the idea and some snippets of code you can use to implement your Polly version of the blog.

#### Step 3 of 6: __get_content_read_by_polly()

At this point, we have all the text we would like to read: the third step is to actually get theme read and persisted on s3. Is as simple as doing the following:

{{< highlight python >}}
def __get_content_read_by_polly(article_path, content):

    # read content
    response = polly_client.synthesize_speech(
        VoiceId='Matthew',
        OutputFormat='mp3', 
        Text = content)

    # save mp3
    with open('speech.mp3', 'wb') as f:
        f.write(response['AudioStream'].read())
    
    # upload mp3
    with open('speech.mp3', 'rb') as f:
        s3_client.upload_fileobj(f, CONTENT_BUCKET, f'mp3/{article_path[:-1]}.mp3')

    return f'mp3/{article_path[:-1]}.mp3'
{{< / highlight >}}

Pretty cool right? But I had this feeling I was missing something, and actually, I was...

#### First Neural Voices TTS

AWS Polly has a [Neural TTS](https://docs.aws.amazon.com/polly/latest/dg/NTTS-main.html) system that can produce even higher quality voices than its standard voices. The NTTS system produces the most natural and human-like text-to-speech voices possible. Standard TTS voices use concatenative synthesis. This method strings together (concatenates) the phonemes of recorded speech, producing very natural-sounding synthesized speech. However, the inevitable variations in speech and the techniques used to segment the waveforms limit the quality of speech.
The Amazon Polly Neural TTS system doesn't use standard concatenative synthesis to produce speech. It has two parts:

- A neural network that converts a sequence of phonemes—the most basic units of language—into a sequence of spectrograms, which are snapshots of the energy levels in different frequency bands
- A vocoder, which converts the spectrograms into a continuous audio signal.

The first component of the neural TTS system is a sequence-to-sequence model. This model doesn’t create its results solely from the corresponding input but also considers how the sequence of the elements of the input works together. The model chooses the spectrograms that it outputs so that their frequency bands emphasize acoustic features that the human brain uses when processing speech. The output of this model then passes to a neural vocoder. This converts the spectrograms into speech waveforms. When trained on the large data sets used to build general-purpose concatenative-synthesis systems, this sequence-to-sequence approach will yield higher-quality, more natural-sounding voices. 

Really. Cool, Amazon. For real. So the question now is... how we can leverage these voices?

#### SSML Specification Language

Our friends from **AWS** have created an entire [Generating Speech from SSML Documents
](https://docs.aws.amazon.com/polly/latest/dg/ssml.html) you can use with Amazon Polly to generate speech. Using SSML-enhanced text gives you additional control over how Amazon Polly generates speech from the text you provide. For example, you can include a long pause within your text, or change the speech rate or pitch. Other options include:

- emphasizing specific words or phrases
- using the phonetic pronunciation
- including breathing sounds
- whispering
- using the Newscaster or Conversational speaking style.

Everything you can do is pretty much collected in this [documentation page](https://docs.aws.amazon.com/polly/latest/dg/supportedtags.html#say-as-tag) so... I'm going to do one step backward.

#### Step 2 (again) of 6: __get_page_content_for_nts()

To let me create SSML tagged speech without dealing too much with regex, markdown, and Python (feel free to do it) - but, at the same time - without dealing too much with HTML and parsing of my *outer text* as well, I put in place a small script to just gather paragraph text. My first intention was to leverage three aspects:

- emphasizing specific words or phrases, for instance the one enclosed in `em` or `strong` tags and replace with SSML notation `<emphasis level="moderate|strong|reduced">`
- put a pause and *breaths* inside my speech;
- let the reading sounds more natural let's say;

So, I decided to replace my old `__get_page_content` with the new method `__get_page_content_for_nts`: it actually just provide the paragraph without losing the `meta` needed for my next step.

{{< highlight python >}}
def __get_page_content_for_nts(url, number_of_words=NUMBER_OF_WORDS, final_sentence=FINAL_SENTENCE):

    # create request
    r = requests.get(url)

    # parse page
    soup = BeautifulSoup(r.text, features="html.parser")

    # get all paragraph
    paragraphs = soup.find("div", {"id": "main"}).findAll("p")

    # accumulate outer text
    page_with_no_code = []

    # for each found paragraph
    for paragraph in paragraphs:
    
        # exclude portion of code
        if paragraph.findAll("div", {"class": "highlight"}):
            continue

        # get the text out of the paragraph
        text = paragraph.text.strip()

        # exclude a common header
        if "Subscribe to my newsletter to be informed about my new blog posts, talks and activities." in text:
            continue

        # accumulate page text
        page_with_no_code.append(paragraph)

    # return result
    return page_with_no_code
{{< / highlight >}}

#### Step 2.5 (new) of 6: __add_SSML_Enhanced_tags()

Ok, we now have paragraphs including tags and many other things we would like to ignore in our parsing - but you can use them to experiments with Polly. The important thing to notice is that not all the features - and not all the voices - are available if you want to use Neural Voices.

<div class="img_container"><img src="https://i.imgur.com/t47bFWY.png" style="width: 100%; marker-top: -10px;"/></div>

Have a look at the right columns to discover more about the features current limitations. So... ok, I will shut-up and provide my script:

{{< highlight python >}}
def __add_SSML_Enhanced_tags(paragraphs):

    # tag to start a speach
    text = "<speak>"
    
    # add informal style
    text = f'{text}<amazon:domain name="conversational"><amazon:effect name="drc">'

    # # add breathing to sounds more natural
    # text = f'{text}<amazon:auto-breaths>'

    # for each paragraph
    for paragraph in paragraphs[:NUMBER_OF_PARAGRAPHS]:
        
        # prepare the paragraph with dot and comma breaks
        paragraph_text = paragraph.text.strip()
        # paragraph_text = paragraph_text.replace("...", "<break time=\"500ms\"/>")
        # paragraph_text = paragraph_text.replace(". ", "<break time=\"800ms\"/>")
        # paragraph_text = paragraph_text.replace(",", "<break time=\"300ms\"/>")

        # prepare the paragraph with slang expression
        paragraph_text = paragraph_text.replace("btw", "<sub alias=\"by the way\">by the way</sub>")
        paragraph_text = paragraph_text.replace("PoC", f"<say-as interpret-as=\"spell-out\">PoC</say-as>")

        # empthatyse em words
        # ems = paragraph.findAll("em")
        # for em in ems:
        #     paragraph_text = paragraph_text.replace(f"{em.text}", f'<emphasis level="moderate">{em.text}</emphasis>')

        # # pronunce strong words loudly 
        # strongs = paragraph.findAll("strong")
        # for strong in strongs:
        #     paragraph_text = paragraph_text.replace(f"{strong.text}", f'<emphasis level="moderate">{strong.text}</emphasis>')

        # print(paragraph)
        # print(paragraph_text)

        # concat paragraph parsed to text
        if len(f"{text} {paragraph_text}") > 1490-len(f" {FINAL_SENTENCE}"):
            break
        else:
            text = f"{text} {paragraph_text}"

    # close the text
    #text = f"{text} {FINAL_SENTENCE}</speak>"
    text = f"{text} {FINAL_SENTENCE}</amazon:effect></amazon:domain></speak>"

    # close the text
    return text
{{< / highlight >}}

As you can see, in the beginning, I created some logic to provide emphasis at a deeper level, even over STRONGS words but then I switched to the conversational domain that just creates the natural kind of speech to the text I wanted to reach - plus some more things, like the `dynamic range compression` (more [here](https://docs.aws.amazon.com/polly/latest/dg/supportedtags.html#drc-tag)) and the reading of some aliases. The **important thing** to notice is that I provide a maximum number of paragraphs (not chars anymore) and, thus, I have to cut my text up to `1500` chars to avoid `TextLengthExceededException`. You can find more about the exception you can get [here](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/polly.html#Polly.Client.synthesize_speech). Also, if you want to have an SSML validator, I found [this one](http://garrettvargas.com/ssml.html) and the [AWS Polly Console](https://eu-west-1.console.aws.amazon.com/polly/home/SynthesizeSpeech?region=eu-west-1) pretty useful.

The result is... we have the text ready to be read, but we need to update `__get_content_read_by_polly` just a bit

#### Step 3 (renewed) of 6: __get_content_read_by_polly()

I decided to use the Matthew Neural Voice and put my mp3 files using the same `folder-structure` I have on my blog, under the mp3 prefix 😎 Here's the code.

{{< highlight python >}}
def __get_content_read_by_polly(article_path, content):

    # read content
    response = polly_client.synthesize_speech(
        Engine='neural',
        LanguageCode='en-US',
        OutputFormat='mp3', 
        Text = content,
        TextType='ssml',
        VoiceId='Matthew'
    )

    # save mp3
    with open('speech.mp3', 'wb') as f:
        f.write(response['AudioStream'].read())
    
    # upload mp3
    with open('speech.mp3', 'rb') as f:
        s3_client.upload_fileobj(f, CONTENT_BUCKET, f'mp3/{article_path[:-1]}.mp3')

    return f'mp3/{article_path[:-1]}.mp3'
{{< / highlight >}}

At this point, we have the whole articles read in s3, and we just need to change all our markdown files. 

#### Step 4 of 6: __get_markdown_list()

Getting my markdown list recursively is just as simple as doing this:

{{< highlight python >}}
def __get_markdown_list(base_path=BASE_PATH):

    # get list of all markdown
    list_of_files = list(Path(base_path).rglob("*.md"))

    # return it
    return list_of_files
{{< / highlight >}}

Before going ahead, I want to preview one more thing: to let mp3 audio be played correctly, I had to change my Hugo theme template to support one more `metadata` in my front-matter. To know more about Hugo and Front Matter, just have a look at [Front Matter](https://gohugo.io/content-management/front-matter/) in the official Hugo documentation. For the laziest, Hugo allows you to add front matter in yaml, toml, or json to your content files. Front matter allows you to keep metadata attached to an instance of a content type—i.e., embedded inside a content file.

Exactly what I want to do!

<div class="img_container"><img src="https://i.imgur.com/lvdjonJ.gif" style="marker-top: -10px;"/></div>

Let's go ahead, and I will show my reasoning at the last step... keep going, we are close to the end (for real XD).

#### Step 5 of 6: __get_markdown_list()

The match is pretty simple: if the end of the audio object prefix path of the s3 object provided by the `__get_content_read_by_polly` is equal to the name of the markdown file, just pair theme to modify the front-matter of that post. I have to be this level of "sure" (exactly equal) due to name collision so my suggestion is ... just put some print if you have a setup like mine, to be sure ;)

{{< highlight python >}}
def __match_audio_and_post(file_list, audio_list):

    # match_dict
    matches = {audio_path : '' for audio_path in audio_list}

    # find match by name
    for audio_path in audio_list:
        for file_name in file_list:
            if audio_path.split("/")[-1].replace(".mp3", "") == str(file_name).split("/")[-1].replace(".md", "").lower():
                matches[audio_path] = str(file_name)
                continue
    
    # return matches
    return matches
{{< / highlight >}}

And finally... modify the all articles in one shot! What could ever happen...

#### Step 6 of 6: __insert_new_audio_reference()

The matches are ready, we just need to transform each front matter like this:

{{< highlight text >}}
---
layout: post
title: "I hacked my blog to let AWS Polly create podcast over it"
date: 2020-11-26
categories:
...
{{< / highlight >}}

in a front matter like this:

{{< highlight text >}}
---
layout: post
title: "I hacked my blog to let AWS Polly create podcast over it"
date: 2020-11-26
polly: https://madeddu.xyz/mp3/aws/hacked-my-blog-polly.mp3
categories:
...
{{< / highlight >}}

And here we are:

{{< highlight python >}}
def __insert_new_audio_reference(matches):

    # for each match
    for audio_name, file_name in matches.items():

        # read the content
        with open(file_name, "r") as f:
            lines = f.readlines()

        # add the line
        lines = lines[0:4]+[f'polly: {BASE_URL}/{audio_name}\n']+lines[4:]

        # write the new content
        with open(file_name, "w") as f:
            for line in lines:
                f.write(line)
{{< / highlight >}}

### One more thing

How can we leverage the new meta inside a Hugo Template? Of course, using the `.Params` field inside the theme used to produce the blog page:

In my case,

{{< highlight html >}}
{{ define "main" }}

	<span>

		<h1>{{ .Title }}</h1>

		<p>Subscribe to <a href="https://tinyletter.com/made2591">my newsletter</a> to be informed about my new blog posts, talks and activities.</p>

		{{ partial "post-meta" . }}
	</span>

	{{ if .Params.polly }}<audio controls style="width: 100%"><source src="{{ .Params.polly }}" type="audio/mpeg"></audio><br />{{ end }}

	<p>
	    {{ .Content }}
	</p>

	<p>Subscribe to <a href="https://tinyletter.com/made2591">my newsletter</a> to be informed about my new blog posts, talks and activities.</p>

    {{ partial "disqus.html" . }}

{{ end }}
{{< / highlight >}}

In this way, you are gonna put an HTML5 `audio` link to the respective content for your article, and you are ready to run your blog as AWS does! 

### Conclusion

The next step is to put everything under an API and use it during my build pipeline to let me produce my mp3 at build-time. [Here](https://gist.github.com/made2591/7ac3a9b6e1212a52aabf925c83a1b719) you can find a gist with the whole script I used - it's already written like a lambda... Oooops 😜😜😜
Finally, in this article I wrote really BAD code as usual, but I hope I provided to you with some insights about how you can leverage AWS Polly to do fancy things and become the best friend of your boss. Just kidding, you can't and you know it.
XOXO

Have fun and stay safe! 🖖