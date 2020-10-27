# Hackathon 2

In this hackathon, you will be building an ASR system and using it to recognize speech in the form of a set of audio recordings of people placing pizza orders and transcriptions of those recordings. You will calculate the word error rate (WER) of a baseline system and try to improve the WER by modifying the pronunciation and language models. You'll be completing this lab independentlyor with one classmate in a team of two.

By 11:59pm on Thursday 10/29, push: 

* a PDF containing your answers to the questions (marked in boldface, beginning with Q)
* your best performing language model (firstname_lastname.lm)
* your best performing lexicon (firstname_lastname.dic file)

As you know, it is common practice to refine your models on a held-out development (*dev*) set of data and to then use a disjoint and unseen test set for final verification of your models. I have provided a development set of recordings and transcriptions. When you have submitted your best grammar and lexicon, I will test your work on a separate test set of recordings and transcriptions that is similar but not identical to the development set given to you. 

The student or students with the lowest WER on the test set will have their second lowest problem set grade dropped when I calculate their final grade.

*Acknowledgement: This lab uses materials generously shared with us by Prof. Alan Black of Carnegie Mellon University.*

## Preliminaries

Today you will learn how to build a speech recognizer using the Carnegie Mellon University Sphinx system, which is a traditional statistical ASR system. Sphinx is not state-of-the-art, but it is easy to use! There are three major components that go into most typical statistical ASR systems:

1. The Acoustic model defines the acoustic units of recognition and the statistical models used to identify them. It is constructed by a training process which takes segmented audio data and the corresponding transcriptions as its input.
2. The Lexicon (or pronunciation model) defines the mapping of words to acoustic units. It is constructed by hand, or by using trained letter to sound rules, or frequently some combination of the two.
3. The Language model (LM, also called a "grammar") defines the set of possible sentences that can be recognized, as well as their relative prior probabilities.

## Part 1: Install the required data and tools

### 1.1 Installation (Linux or Mac)
After you have cloned your repository, On Linux or Max OSX, follow these commands below in a terinal:
      
```
      cd hackathon2
      tar zxf pocketsphinx-0.5-20080912.tar.gz
      cd pocketsphinx-0.5
      ./build_all_static.sh
      cd ../hackaton2_data
      ../pocketsphinx-0.5/pocketsphinx/scripts/setup_sphinx.pl -task rm     
```

### 1.2 Installing on Windows?

If you are using the Windows 10 unix option (which you should be doing by now in your CS career), you should be able to follow the instructions above.

If you are a Windows user who works from a DOS command line, the executable programs (`.exe` files) needed are included in the `hackathon2_data\bin` directory. You will need to have Perl installed (e.g., Active Perl). You should verify that you have a working version of Perl before proceeding by opening your DOS command line, navigating to the `hackathon2_data` directory, and running:

      perl scripts_pl\00.verify\verify_all.pl 
    
When I was last able to test this, I was able to call the commands shown in Part 2 through the end from the DOS cmd. When you call a binary (i.e., something that doesn't have a file extension in the commands below), you'll probably need to add `.exe`. And when you call a perl script (ending in `.pl`), you might need to preface the command with the word perl.

## Part 2: Build the baseline system

### 2.1 Acoustic Model

Training an acoustic model takes a long time, which makes experimenting with model parameters a time-consuming task. For this reason, I have trained the acoustic model for you. The acoustic model files are in the `model_parameters` and `model_architecture` directories.

Look at the files in the `model_architecture` directory and answer the following questions.

**Q1: How many phones are used in the acoustic model?**

**Q2: Are there any phones missing that you might have expected to see given the set of Arpabet symbols we discussed? Are there any additional phones that we did not talk about in class? What do you think the additional phone(s) represent?**

**Q3: Some of these files contain lists of triphones. Look at the headers in one of these files to get a sense of what the different fields are, then tell me (1) what triphone this represents, and (2) give an example of a word containing this triphone:**

AE   K   B b    n/a    1 2500 2501 2502 2503 2504    N

### 2.2 Lexicon (Pronunciation Model)

You will need to be able to recognize the sorts of words that will be used in speech-driven pizza ordering system. The recordings to be recognized consist of requests for a specific size of pizza with one or more toppings. Each request can optionally start with one of the following phrases:

* I'd like to order a ...
* I want to order a ...
*  Give me a ...

This is followed by the size of pizza requested, either "small", "medium", "large", or "extra large", the word "pizza", and an optional list of toppings. The available toppings are: extra cheese, mushrooms, onions, black olives, green olives, pineapple, green peppers, broccoli, tomatoes, spinach, anchovies, sausage, pepperoni, ham, bacon

I have provided a lexicon file `baseline.corpus.dic` containing all of the above words with one pronunciation each. This file will be used in your baseline system.

### 2.3 n-gram Language Model (a.k.a. "grammar")

To build an n-gram language model, you need a sufficiently large sample of representative text. In real-world systems this is usually collected from digital text resources such as newswires and web sites, or it is created by transcribing speech data. In this case, you will be creating it from scratch. Roughly, you need to create a text file which has many examples of the way people can order pizzas within the limits of the pizza ordering language described above. This text file should be a plain text file consisting of one sentence per line with beginning and end of sentence tags. Punctuation is not needed.

I have given you a very tiny corpus in the `hackathon2_data` directory, called `baseline.corpus`. You'll see that it's just a list of sentences, one per line, each beginning with `<s>` and ending with `</s>`.

**NOTE: When you open this file and edit it with an application like TextEdit, it will often try to save it as rich text or UTF-16. If this happens, it's likely that you'll get errors later on. If you can, use a real text editor (e.g., emacs, vim) or open the files needed for this activity in an IDE that will not automatically convert everything to rtf or UTF-16.**

You can build a trigram language model from this corpus using the  `quick_lm.pl` script in the `hackathon2_data` directory, as follows:

```
      perl quick_lm.pl -s baseline.corpus    
```    

This will create a language model file called `baseline.corpus.lm` and a lexicon called `baseline.corpus.dic`. (Ignore the warnings. You'll know it worked if the last line of output says "Language model completed".)

## Part 3: Run the baseline system

Now you should run the speech recognizer in "batch mode" to test the models that you have built. This requires you to have, in addition to the acoustic model, language model, and dictionary, the following files (which I have provided for you):

* A control file listing the names of the files you wish to recognize, minus the base directory and file extension. Use `pizza_devel.fileids` in the `lab2_data/etc directory`.
* A configuration file containing the command-line arguments for the recognizer. To see the list of all possible options, you can simply run the recognizer (bin/pocketsphinx.batch on Unix/Mac OSX or bin\pocketsphinx_batch.exe on Windows) with no arguments. I have created one for you called baseline.cfg.

These files I have provided are are both ready to go for this example, so you should be able to run the recognizer without changing anything, like this:

```
    bin/pocketsphinx_batch baseline.cfg     
```

Remember that if you're running under Windows from a DOS command line, you might need to add `.exe` to the command.

If recognition was successful, you should see a lot of output on the screen, ending with a few lines like this (the numbers will be different on your computer):

```
INFO: batch.c(386): TOTAL 409.99 seconds speech, 10.60 seconds CPU, 10.67 seconds wall
INFO: batch.c(388): AVERAGE 0.03 xRT (CPU), 0.03 xRT (elapsed)
``` 

You're going to have to change the configuration (`.cfg`) file later, so here's an overview of its format and contents:

```
-hmm followed by the acoustic model directory
-dict followed by the lexicon file
-lm followed by the language model
-ctl followed by the control file
-adcin yes (tells the recognizer that input is audio, not feature files)
-adchdr 44 (tells the recognizer to skip the 44-byte RIFF header)
-cepext .wav (tells the recognizer input files have the .wav extension)
-cepdir wav (tells the recognizer input files are in the 'wav' directory)
-hyp followed by the output transcription file
-outlatdir followed by the output directory for word lattices
Have a look at the configuration file to see what files you just used to build your model. Remember that all filenames in the configuration file are interpreted relative to the current directory.
```

## Part 4: Testing your reecognizer

The recognition results can now be found in the file `pizza_devel.hyp`. You will now compute the word error rate (WER), which is the standard method for evaluating speech recognition systems. The script `word_align.pl` in the `scripts_pl/decode` directory compares the reference transcription to the recognition results and reports the error rate for each sentence, followed by the overall error rate. You can run it like this:

```
    perl scripts_pl/decode/word_align.pl etc/pizza_devel.transcription pizza_devel.hyp     
```

The final three lines of its output will report the number of errors and the error rate.

**Q5: What is the baseline WER?**

## Part 5: Improving the baseline system

### 5.1 Improving the lexicon (pronunciation model)

Take a look at the lexicon file, baseline.corpus.dic. You'll see that there is one word that has two pronunciations: "AND".

AND     AH N D
AND(2)  AE N D 

There are other words in the lexicon that can have multiple pronunciations, depending on personal preference, rate of speech, or dialect. Add some additional pronunciations to the lexicon file, remembering to add them one per line and to use the special numbering notation.

Re-run the recognizer using your improved lexicon file.

    bin/pocketsphinx_batch baseline.cfg     
    perl scripts_pl/decode/word_align.pl etc/pizza_devel.transcription pizza_devel.hyp     
    
**Q6: What is your new WER?**

**Q7: Give a few examples of the pronunciations you added and explain why you think they might have helped to improve perfomance (if they did).**


### 5.2 Improving the the language model

As you look at the output of the testing, you'll see how your results compare to the correct transcriptions. You will notice that there are words in the development data that do not appear in your lexicon and probably some structures that do not appear in your baseline corpus.

Make a copy of the `baseline.corpus` file called `larger.corpus` and add an additional 10 sentences, keeping in mind some of the missing words and structures. Then regenerate your language model in the same way you created it above. 

Now alter your configuration file (or create a new one) that includes the names of your newly created lm and lexicon. Then run the recognizer with that configuration file.

**Q8: What is your new WER?**

**Q9: Is your new lexicon longer than the baseline lexicon? Why?**

**Q10: Visually inspect and compare the two .lm files. What differences do you see?**


### 5.3 Degrading performance in a few easy steps!

Go to news.google.com, select an article that interests you, and copy the first ten sentences into a text file, one sentence per line. Convert the file to all uppercase and make sure to insert `<s>` and `</s>` at the beginning and end of each line.

    tr '[:lower:]' '[:upper:]' < input.txt > output.txt     
       

Then regenerate your language model and lexicon in the same way as above. Alter your configuration file (or create a new one), and run the recognizer with your new grammar and new lexicon built on the random article from news.google.com.

**Q11: What is your new WER?**

**Q12: Why do you think it's so much worse?**

At this point, you have likely improved somewhat over the baseline provided. If you're happy with earning a "meets expectations" (8 out of 10 points) on this assignment, simply turn in your best `.lm` and `.dic` files, along with a pdf containing answers to questions 1-12 above.

If you're having fun and would like to learn more and want a chance at "exceeds expectations" (9 out of 10) or even "outstanding" (10 out of 10), continue on.

## Part 6: Continuing to improve performance

### Option 1: Improving your existing models

Here are some possibilities:

* Continue adding as many sentences to your corpus as you can, either manually or using a script that generates new sentences based on the patterns you observe in the data.
* Look at the output of your recognizer and strategically add new sentences to your corpus.
* Listen to the recordings and strategically add new pronunciations to your lexicon.
* Build a language model using fancier tools (srilm, OpenGRM, kenlm) on a very large corpus. (The quick_lm.pl script is not designed to handle larger corpora, and so you should not try to use it on more than a few thousand sentences.)

**Q13 (if seeking 9/10 or 10/10): What is your very best WER?#**
