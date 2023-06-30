## Loop to read multiple sequences with AlphaFold 2 Colab

This loop must be inserted before the execution of the AlphaFold2 Colab predictions. Instead of just inserting a sequence for its prediction, it allows inserting a list with any number of sequences

The list with sequences must have the following structure:

```{python}
>>XP_001388503.1
MEYPSKQYPVPAGVHIIPEHLLDLRPDSEVDYDLLHPRPVTDEKNIWLFWHSGYSTMHPYTKRNVRAWHRRFSKAGWIVR

>>XP_001388543.2
MAAMMTPGSPVRLGPGPRRRSSNLVILLLVIIALLWSLVIHQNIGRGLHVTLDDIRDPLDAIVNNTLGFEKIFAISPAQR
```

```{python}
from google.colab import files
import os
import re
import hashlib
import random

from sys import version_info 
python_version = f"{version_info.major}.{version_info.minor}"

def add_hash(x,y):
  return x+"_"+hashlib.sha1(y.encode()).hexdigest()[:5]

# upload input file
uploaded = files.upload()

# extract sequences from input file
sequences = []
current_seq = ''
current_id = ''
for line in uploaded[list(uploaded.keys())[0]].decode().splitlines():
    if line.startswith('>'):
        if current_seq != '':
            sequences.append((current_id, current_seq))
            current_seq = ''
        current_id = line.strip().replace('>','')
    else:
        current_seq += line.strip().replace(' ','')
sequences.append((current_id, current_seq))

# create jobname based on input file name
input_filename = list(uploaded.keys())[0]
jobname = os.path.splitext(input_filename)[0]

# remove illegal characters from jobname
basejobname = "".join(jobname.split())
basejobname = re.sub(r'\W+', '', basejobname)
jobname = add_hash(basejobname, ''.join([seq[1] for seq in sequences]))

# check if directory with jobname exists
def check(folder):
  if os.path.exists(folder):
    return False
  else:
    return True
if not check(jobname):
  n = 0
  while not check(f"{jobname}_{n}"): n += 1
  jobname = f"{jobname}_{n}"

# make directory to save results
os.makedirs(jobname, exist_ok=True)

# save queries
queries_path = os.path.join(jobname, f"{jobname}.csv")
with open(queries_path, "w") as text_file:
  text_file.write("id,sequence\n")
  for i, seq in enumerate(sequences):
    text_file.write(f"{seq[0]},{seq[1]}\n")

print("jobname",jobname)
print("num_sequences",len(sequences))

```

## EXPLANATION OF THE CODE BY PARTS

In this section, the necessary libraries for the operation of the program are imported, such as Google Colab files to be able to upload the input file, os for the creation of directories and files, re for the manipulation of regular expressions and hashlib for the calculation of SHA1 hashes. In addition, the add_hash function is defined which adds a SHA1 hash of the arguments x and y to the string x.

```{python}
from google.colab import files
import os
import re
import hashlib
import random

from sys import version_info 
python_version = f"{version_info.major}.{version_info.minor}"

def add_hash(x,y):
  return x+"_"+hashlib.sha1(y.encode()).hexdigest()[:5]

```


### Input file upload:
This line of code uploads the input file in the Google Colab environment and saves it in the uploaded variable.
```{python}
uploaded = files.upload()

```

### Extracting the sequences from the input file:
This section extracts the sequences from the input file, line by line. First, an empty list sequences is initialized to hold the sequences, and an empty string current_seq to hold the current sequence being read. Then, it iterates over each line of the input file, and if the line starts with '>', it means that it is a new sequence and it saves the old sequence in the sequences list and restarts the current_seq chain. If the line does not start with '>', the line is added to the current string, removing trailing and trailing spaces from the line.

```{python}
sequences = []
current_seq = ''
for line in uploaded[list(uploaded.keys())[0]].decode().splitlines():
    if line.startswith('>') and current_seq != '':
        sequences.append(current_seq)
        current_seq = ''
    current_seq += line.strip().replace(' ','')
sequences.append(current_seq)

```

### Creating the job name and checking its existence:
In this section you create a name for the job to be performed. First the input file name is extracted and the extension is removed with os.path.splitext(). Then, spaces are removed from the name with " ".join() and disallowed characters are removed with re.sub(). Finally, a SHA1 hash of the concatenation of the sequence names is added to the resulting string, and stored in the jobname variable. If the directory with the name jobname already exists, a sequential number is added to it.

```{python}
input_filename = list(uploaded.keys())[0]
jobname = os.path.splitext(input_filename)[0]

basejobname = "".join(jobname.split())
basejobname = re.sub(r'\W+', '', basejobname)
jobname = add_hash(basejobname, ''.join(sequences))

def check(folder):
  if os.path.exists(folder):
    return False
  else:
    return True

if not check(jobname):
  n = 0
  while not check(f"{jobname}_{n}"): n += 1
  jobname = f"{jobname}_{n}"

```

### Save queries
This code has two parts: the first creates a directory to save the results and the second saves the protein sequences to a CSV file within that directory. Here is a more detailed explanation:

1 - "os.makedirs(jobname, exist_ok=True)" creates a directory with the name specified in the jobname variable. If the directory already exists, it does nothing (exist_ok=True). This directory will be used to save the results.

2 - "queries_path = os.path.join(jobname, f"{jobname}.csv")" creates a path for the CSV file to be saved in the directory created in the previous step. The path is a concatenation of the directory name (jobname) and the CSV file name (also jobname, but with a .csv extension).

3 - "with open(queries_path, "w") as text_file:" opens the CSV file in write mode ("w") and assigns it to the text_file variable. The with clause is a way to open a file and ensure that it is closed gracefully when finished using it.

4 - "text_file.write("id,sequence\n")" writes the first line of the CSV file, containing the id and sequence column headers.

5 - "for i, seq in enumerate(sequences):" loops through each protein sequence in the sequences list and assigns it to the seq variable. The variable i is a counter that goes from 0 to the length of sequences.

6 - "text_file.write(f"{seq[0]},{seq[1]}\n")" writes a line to the CSV file for each protein sequence in sequences. Each line contains the sequence id (the sequence number, obtained from counter i) and the protein sequence itself. The sequence is in seq[1], since the seq variable is a tuple containing both the id and the sequence.

6 - "print("jobname",jobname)" and "print("num_sequences",len(sequences))" print the name of the directory where the results were saved and the number of protein sequences that were saved, respectively.
```{python}
# save queries
queries_path = os.path.join(jobname, f"{jobname}.csv")
with open(queries_path, "w") as text_file:
  text_file.write("id,sequence\n")
  for i, seq in enumerate(sequences):
    text_file.write(f"{seq[0]},{seq[1]}\n")

print("jobname",jobname)
print("num_sequences",len(sequences))
```

