# Improving and automating testing using the LMs 

What do I want to achieve?
- Explore how can language models be used for generating unit tests
- Learn something new
- Ideally: language model running locally and generating tests given problem statement and code. 


## Data

I've focused on the competitive problems for the following reasons:
  - It's easy to get
  - There are example tests
  - Solutions are short

There are some disadvantages though:
  - Code quality is poor
  - Statements often include non-technical information (legend)
  - The problems are usually hard

I've also considered using code from the small-sized operating system: 
  - Lots of well documented and carefully designed C code
  - Too difficult for me to parse

---

Even though we won't need many - I've decided to implement the scraper [./scrape.ipynb](./scrape.ipynb)

Scraper does not work with the solutions - filtering by the verdict and number of tests passed complicates things (issues with the session tokens). 

The statements scraping works as expected, though/*. 

\**(unless you want to load more than 1 page of problems)*

In the end - we have 20 problem [statements](./scraped/statements/), each having a
  - Statement
  - Example tests
  - Time and memory limits

In addition to the Codeforces ones, I've also downloaded a couple statements from the [ACMP](acmp.ru). 
The ACMP-ones are simpler and shorter.

--- 

As it turns out - the real competitive problems have long descriptions and are too complicated for the small LM
to understand. That's why I've obtained 60 more ultra-simple problem (one-liners)

## State of the Art

How do **large** language models handle such problems?

I've performed simple tests on this [contest](https://codeforces.com/contest/2065).


**o3-mini** generates correct tests input for nearly all the problems. (The prompt was as simple as possible: problem statement + ```write two syntactically correct tests and provide the outputs. 
Do not write anything other that the tests.```)

Majority of generated tests had both correct input and the output even for the non-trivial D and E problems.

**4o** provides syntactically correct tests that have ~60% of the outputs correct. 
(Tested only for the E problem).

It can be concluded, that **large** work pretty well.


## What did I implement?


### GPT 2

First thing that came to mind: paste the problem statement, asking model to generate the test. 
This approach **does not work at all**. Problem statements are too long to fit in the context (~20% of the statements were skipped). 

The model was not trained for the
user and assistant scenario - promts that try to make use of it fail completely. Interesting observation: model rather repeats the statement than write tests: [samples](./generated/gpt2-1/).

Simpler prompt (`statement + "\n\nInput:"`) gave better results:
  - Better test structure
  - Uses digits
  - Nevertheless, **nonsence**

Model suffers heavily from the repetition and lack of the understanding: [samples](./generated/gpt2-4/).

There is a "repetition penalty" hyperparameter - sounds promising, but the implementation is fairly simple.

It does not consider the words position and number of occurences. Coupled with the fact that tests are mostly made of the numbers and have fixed structure, it resulted in the worse results. Generated texts consisted of a bunch of unique, but irrelevant tokens: [samples](./generated/gpt2-5/).

### Other models

Other than the GPT2, I've tried using GPT2-XL, CodeGPT and bigcode/starcoder. 

- XL model : a bit better test structure, somehow resembles the original problem. Still, suffers from the 
  repetition: [samples](./generated/gpt2-6/).

For the code-based models, I've tried running a simple test: generate the rest of the factorial function,
given the 80% of solution and a comment.

- CodeGPT has failed completely
- StarCoder did not finish the factorial function, but started to write the one to calculate binomial coefficients (the inference took 5 hours :) )

### Simple tests

It became quite obvious that some simpler problems are required as the larger ones confuse the model.
Experimenting with the simpler problems is implemented in the [gpt2-simple.ipynb](./gpt2-simple.ipynb).

One cool thing about the small tests - they can be processed in batches (assuming that you have GPU). 
As it turns out, it works pretty well on the Apple Macbook pseudo-GPU's, making the process ~60 times faster.

### Embeddings

I've also explored if there is some obvious corelation between the statements and the embeddings: 
[embeddings.ipynb](./embeddings.ipynb).

I've considered:
  - Problems with the same statement, but different tests
  - Problems with different statements, but same tests
  - Different problems on different topics

As expected, we can see from the embeddings that both statement and tests affect the embeddings. 
When considering problems based on the different topics - mild separation can be noticed. 

## File structure

`unused/` - LLama (too heavy to run on the laptop) and OSS Operating System that I was planning to use

`generated/` - Tests generated by the LM. Each folder represents a different approach
  - `generated/gpt2-0`: GPT2-small with user / agent prompting
  - `generated/gpt2-1`: GPT2-small with user / agent prompting
  - `generated/gpt2-3`: GPT2-medium with natural prompting
  - `generated/gpt2-4`: GPT2-medium with natural prompting
  - `generated/gpt2-5`: GPT2-medium with the repetition penalty hyperparameter.
  - `generated/gpt2-6`: GPT2-xl with natural prompting
  - `generated/gpt2-simple-0`: GPT2-small with natural prompting

\**0-6 use Codeforces statements for generation*

\**simle use simple statements for generation*

`scraped/` - Problem statements obtained.

`scraped/statements/` - Codeforces statements (too long to obtain sensible results)

`scraped/acmp/` - ACMP statements (shorter and simpler that the codeforces ones, but still - too long to obtain sensible results)

`scraped/simple/` - One-liners, formatted specifically for easier parsing

`gpt2.ipynb` - Generating tests using the GPT2

`code.ipynb` - Generating tests using the code-based models

## Further reading and interesting facts

Here are couple articles that I've found particulary interesting:

#### [**GPT o1 overview**](https://openai.com/index/learning-to-reason-with-llms/)
 Main take: given the significant compute and 10 tries 
(vs 2 tries for humans), LM has achieved the 1800 elo at the Codeforces website.
(During the last year of school my rating was at around 1900).


#### [**GPT o1 overview - more technical information and benchmarks**](https://cdn.openai.com/o1-system-card.pdf#page=16)
As it turns out, the models are heavily tested not only on the generally 
unethical prompts (i.e. harassing, propaganda, etc), 
but also for the **computer security, biological security (i.e. how to make a poison), persuasion** and then are tuned down, in order to perform worse on such tasks.

---
Author: Denys Zinoviev 341946, 10.02.25
