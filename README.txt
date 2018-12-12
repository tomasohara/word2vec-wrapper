This directory (project) contains a python script for interfacing with Gensim's
implementation of Google's word2vec algorithm. See
    https://radimrehurek.com/gensim/models/word2vec.html
This also includes two utility modules. These scripts were developed by me to
support work on a variety NLP-related consulting projects, such as for essay
grading (e.g., for Intemass) and for Internet job search (e.g., for Juju). This soon will be integrated into a broader package for text-analytics, so this should just be considered as an interim project, especially by anyone using it for production work. It is very released as is, just to provide a simple was to word2vc in Python. (There are not unit tests, but trying to reproduce example below should suffice for verification purposes.)

The package requirements are just those for gensim (e.g., numpy, scipy, boto, and six). See https://radimrehurek.com/gensim/install.html.

The software is license under the GNU Lesser General Public Version 3 (LGPLv3). See LICENSE.txt.

Tom O'Hara (tomasohara@gmail.com)
December 2018

Copyright (c) 2012-2018 Thomas P. O'Hara

--------------------------------------------------------------------------------

Example processing (assuming Bash under Linux)

Notes:
- Assumes ~/nltk_data/corpora contains NLTK corpus data (e.g., Reuters and State of the Union)
- Command bracketed by '$: {' and '}' should be pasted into Bash terminal window.
- The results are placed in /tmp/state-of-the-union and /tmp/reuters-samples
- The text is not preprocessed, so some tokens with punctuation are output.

1. Isolate text files from corpora and put in /tmp directories. The google_word2vec script doesn't process subdirectories, so the text files to be processed as put in the base directory (e.g., './state_union/*.txt' => './').

$: {
   export PYTHONPATH=".:$PYTHONPATH"
   export NLTK_DATA=$HOME/nltk_data
   
   # Use all of State of the Union
   base=/tmp/state-of-the-union
   mkdir -p $base
   unzip "$NLTK_DATA/corpora/state_union.zip" -d $base
   mv -iv $base/state_union/*.txt $base
   num_words=$(cat $base/*.txt | wc -w)
   echo "$num_words words in State of Union corpus ($base)"

   # Use test portion of Reuter's corpus
   base=/tmp/reuters-sample
   unzip -j "$NLTK_DATA/corpora/reuters.zip" "reuters/test/*" -d $base
   echo "$num_words words in sample of Reuter's corpus ($base)"
}

2. Run word2vec over text files producing separate models for State of the Union and Reuter's corpus samples. Each run takes about 20 minutes on a 2-CPU Ubuntu VM with 8GB memory running on an Intel i7 quad-core machine. The output models are 5M+ bytes each.

$: {
   base=/tmp/state-of-the-union
   python -m google_word2vec --save $base >| $base.log 2>&1
   base=/tmp/reuters-sample
   python -m google_word2vec --save $base >| $base.log 2>&1
}

3. Show similarity data for a few sample words for each model. (To avoid exceptions, make sure the words occur in the corpora.)

$ echo "no new taxes" | ./google_word2vec.py --load --show-similarity /tmp/state-of-the-union.word2vec
=>
...
no: there: 0.975, much: 0.962, only: 0.949, but: 0.949, Nothing: 0.949
new: $40: 0.939, funding: 0.932, tax: 0.930, provide: 0.927, increase: 0.927
taxes: ensuring: 0.996, non-nuclear: 0.996, illegal: 0.996, hospitals: 0.995, imports: 0.995

$ echo "no new taxes" | ./google_word2vec.py --load --show-similarity /tmp/reuters-sample.word2vec
=>
...
no: there: 0.983, we: 0.982, that: 0.982, "The: 0.981, "We: 0.978
new: merger: 0.989, cut: 0.986, acquisition: 0.984, held: 0.982, proposed: 0.982
taxes: begin: 0.995, Ansett: 0.994, real: 0.993, benefit: 0.993, improve: 0.993
