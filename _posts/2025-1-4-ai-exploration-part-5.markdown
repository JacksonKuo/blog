---
layout: post
title: "AI Exploration: Part V - Image Recognition"
date: 2025-1-4
tags: ["ai"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement

Verify OpenAI Vision's image recognition capabilities.[^1] The test will check if Vision can compare two images. The first image being a company logo and the second image a homepage screenshot. Can Vision confirm the screenshot has an instance of the logo? 

# Approach

I used a stock Amazon logo and a screenshot of the amazon.com homepage. 

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/logo-2.png){: width="250"}
{: refdef}
{:refdef: style="text-align: center;"}
\[Stock Amazon Logo\]
{: refdef}

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/site-2.png){: width="800"}
{: refdef}
{:refdef: style="text-align: center;"}
\[Amazon Homepage Screenshot\]
{: refdef}

A couple of important things to note:
* Vision can't parse PDFs, only images with the following extensions: `.png`, `.jpeg`, `.webp` , and non-animated `.gif`
* Vision can't directly compare two images
* The Amazon homepage screenshot does not have an logo. The usual top section was not included within the image snip

My approach was to first ask OpenAI to describe any logos in the images and then secondly ask if one image could be found in the other. 

```python
import base64
from openai import OpenAI

client = OpenAI()

def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode("utf-8")

logo_path = "logo-2.png"
base64_logo = encode_image(logo_path)

screenshot_path = "site-2.png"
base64_screenshot = encode_image(screenshot_path)

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{
        "role": "user",
        "content": [{
            "type": "text",
            "text": "only describe any logos in the images. does one image make an appearance in the other image? and if so, where? Do not guess",
        },
        {
            "type": "image_url",
            "image_url": {
                "url": f"data:image/png;base64,{base64_logo}", 
                "detail": "low"},  
        },
        {
            "type": "image_url",
            "image_url": {
                "url": f"data:image/png;base64,{base64_screenshot}", 
                "detail": "low"},
        },],}
    ],
    max_tokens=300,
)

#Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='The first image features the Amazon logo, which includes the word "amazon" in dark letters with an orange arrow underneath that runs from the letter "a" to the letter "z."\n\nIn the second image, the Amazon logo appears again in the top left corner, integrated into the website design as part of the header.', refusal=None, role='assistant', audio=None, function_call=None, tool_calls=None))
```

# Results

Vision appears to not have a great concept of logos. The output states, `the Amazon logo appears again in the top left corner`, however that is clearly not true. Perhaps Vision is getting confused with the text `Amazon Basics` in the top bar. And maybe the "Amazon" in `Amazon Basics` text is being considered a logo. 

The comparsion is successful on this next image which has no Amazon textwords. 

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/site-3.png){: width="800"}
{: refdef}
{:refdef: style="text-align: center;"}
\[Amazon Homepage Screenshot with no Amazon text\]
{: refdef}

If a logo is mostly text, Vision may mistakenly associate regular text words to that logo. 

# Costs

The cost calculation is still a bit of a mystery to me. I'm not sure if the image base64 counts as part of the input tokens or if "low res" mode only charages a flat 85 token plus output tokens. I'll continue to test and update this section, but i'm seeing input tokens in the tens of thosands per image right now

* Don't forget to specify "low res" mode in the image url:
    * `image_url": {"url": f"data:image/png;base64,{base64_screenshot}", "detail": "low"},`
* Add a max token cap
    * `max_tokens=300`

The following is a pricing table according to the Vision pricing calculator[^2]

| Resolution | Resize | Cost | Tokens |
|---|---|---|---|
| High res | 2048 x 2048 | $0.003825 | 25501 |
| Low res | 512 x 512 | $0.000425 | 2833 |
| Sample | 2428 × 1468 | $0.005525 | 36835 |

I'll need to research how this compares with something like AWS Rekognition. 

# References

[^1]: [https://platform.openai.com/docs/guides/vision](https://platform.openai.com/docs/guides/vision)

[^2]: [https://openai.com/api/pricing/](https://openai.com/api/pricing/)