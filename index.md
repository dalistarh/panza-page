A personal assistant is a software application that helps manage various daily tasks such as email, calendar, writing, summarizing, recalling past events or actions, etc, that take up unneeded time in today’s laptop and phone environments. Most personal assistants are based on an LLM, and typically on an LLM residing in the cloud and accessed via an API. This has the advantage of simplicity and the ability to use the capabilities of the latest large models, but makes it computationally expensive and difficult to:

- **Personalize** the model, that is, customize it to the specific individual. 
- **Protect the user’s privacy**, that is, allow the AI to have access to very personal information of a caliber that - unlike corporate data - many people will never agree to share even if promised that this will be protected by the provider’s cloud. 
- **Have 0 attitude**: generate anything, allowing the user to make their own copyright and morality decisions, as any individual does, aafter the requested content is generated. 

Panza is a desktop LLM-based assistant that aims to overcome these three problems. It is the user’s confidant, having secure and comfort-inspiring access to all data in the user’s environment without developing an attitude, i. e. without attempting to control their creativity and communication style. In our vision, Panza will continuously improve based on the data of a specific person, learning the history, the associativity patterns, and the style of the user, and using this data to deliver a great individual support experience.

Our first version of Panza is Panza Mail.

## What is Panza Mail?
Panza Mail is a tool for automatically generating emails that are customized to your writing style and past e-mail history.
More precisely, we provide process for using your past emails to finetune a publicly-available Large Language Model with 7 billion parameters (we use Mistral-Instruct-7b).
One thing that is special about Panza is that it **can be trained (and run) on a single GPU with
about 22GiB of memory**. This assures privacy, as, generally, there is no need to upload sensitive email data to an external tool or server.
Training is also quick - for a dataset of about 1000 emails, training takes well under an hour. The resulting model largely learns the writing style of the original author.


> [INST] Write a message to my local neighborhood mailing list, asking if anyone can water my plants and feed my cat Spike the last week in March.
> [/INST]

Panza:

>  Hi everyone,
>
> I'll be away from March 23 to March 30. If anyone can water my plants and
> feed my cat Spike, that would be great.
>
> Thanks,
>
> Jen

Mistral-7B-Instruct:

>Dear Neighbors,
>
>I hope this message finds you well. I am writing to kindly request your help with taking care of my plants and cat Spike for the last week in March. I will be away on a business >trip and will not be able to come back home to take care of my garden and my furry friend.
>
>If anyone is willing and able to help me out, I would be extremely grateful. Please let me know if you can come by my house and water my plants and feed Spike. I will make sure to leave you with the necessary instructions and supplies.
>
>Thank you in advance for your kindness and generosity. I look forward to hearing from you soon.
>
>Best regards,
>[Your Name]


As an additional bonus, Panza retains many of the capabilities of the original model, allowing for the injection of world knowledge into the generated emails (however, use at your own risk).


>[INST] Write an email to my friend Stew explaining how a steam engine works.
> [/INST]
>
> Hi Stew,
>
>Steam engines work by heating water to create steam. The steam is
>then used to push a piston, which in turn moves a crankshaft. The
>crankshaft is connected to the wheel, so when the piston moves, the
>wheel moves, and the train moves.
>
>Best,
>
>Jen


## How it works
For most email clients, it is possible to download a user's past emails in a machine-friendly .mbox format.
These are used as training data, as follows. For each email, Panza uses a pretrained model (we use [Mistral-Instruct-7b](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2) with no additional training) to write a command that might lead an LLM to generate the email. This gives us a (semi-synthetic) training set that we use to finetune a LLM to generate emails that match those written by the user, in response to the synthetic prompts created in the previous step.

We then use parameter-efficient finetuning to train the LLM on this dataset. We found that we get the best results with the [RoSA method](https://arxiv.org/pdf/2401.04679.pdf), which combines low-rank (LoRA) and sparse finetuning. If parameter efficiency is not a concern, then regular, full-rank/full-parameter finetuning can also be used. We find that a moderate amount of further training strikes the right balance between matching the writer's style without memorizing irrelevant details in past emails.

Optionally, Panza can be run with a Retrieval-Augmented Generation (RAG) module. This functionality stores past emails in a database and provides several of the most relevant ones for each new query. This allows Panza to better insert specific details, such as a writer's contact information or frequently used Zoom links.


## Try it out

To train your personalized email assistant, please go to the [code](https://github.com/IST-DASLab/panza-dev/tree/master) and follow the instructions in the README. You will need:
* Your emails, exported to `mbox` format.
* A computer, preferably with an NVIDIA GPU with at least 22 GiB of memory. Panza can also be trained on CPU, but this takes considerably longer.
* Basic Python/unix knowledge, such as building environments and running python scripts.
* If you plan to use the Mistral models (or most other viable alternatives), a HuggingFace account.

## Authors

Panza was conceived by Nir Shavit and Dan Alistarh and built by the [Distributed Algorithms and Systems group](https://ist.ac.at/en/research/alistarh-group/) at IST Austria. The contributors are (in alphabetical order):

Dan Alistarh, Eugenia Iofinova, Eldar Kurtic, Armand Nicolicioiu, Mahdi Nikdan, Andrei Panferov, and Nir Shavit.

We thank our collaborators Michael Goin and Tony Wang at NeuralMagic and MIT for their helpful testing and feedback.
