.. title: Adobe password leak analyzed with pandas and Python
.. slug: adobe-password-leak-analyzed-with-pandas-and-python
.. date: 2015-04-15 22:38:53 UTC+02:00
.. tags: python, pandas, security
.. category:
.. link:
.. description:
.. type: text

I recently found out that one of my throwaway email-accounts was in the Adobe password leak. I wanted to see what was out there and downloaded the leaked data. It looks like this:

::

	<some ID>-|-<username, mostly missing>-|-<email>-|-<password hash>-|-<password hint>|--
	103238705-|--|-asdgagr@yahoo.com-|-BB4e6X+b2xLioxG6CatHBw==-|-boyfriend|--
	103238706-|--|-fasggfso@hotmail.com-|-Cm8mAzxAiwzioxG6CatHBw==-|-dance|--
	103238707-|--|-witdfhadki@gmail.com-|-n+TZlu41zyHioxG6CatHBw==-|-|--
	103238708-|--|-isdfahdh8@gmail.com-|-FAniAwP+U13ioxG6CatHBw==-|-|--
	103238709-|--|-ojaadf@yahoo.com-|-kxiV+a47bSlf+E5Ulu/AzA==-|-newest|--
	103238710-|--|-sahjg@hotmail.com-|-UimSy9NunUU=-|-dog|--

Here's the relevant `xkcd <http://xkcd.com/1286/>`__

.. figure:: http://imgs.xkcd.com/comics/encryptic.png
   :alt: xkcd encryptic
   :align: center
   
.. TEASER_END

I put a small script together to convert the file to a tab-separated file. This is probably not necessary, but I didn't have to think much about it and it would be easier later on. I do all this with in an `Anaconda 3.4 <http://docs.continuum.io/anaconda/>`__ installation.

.. code:: python

	import csv
	from io import open
	filename = 'cred.csv'
	userDict={'idnumber':[],
	          'username':[],
	          'emailfront':[],
	          'emailprovider':[],
	          'passwordhash':[],
	          'hint':[]}
	rowsList = []
	with open(filename, "rt") as in_file:
	    with open('converted.tsv','w', newline='') as out_file:
	        csvwriter=csv.writer(out_file, delimiter='\t')
	        for i in in_file:
	            try:
	                text = in_file.readline()
	                temp = []
	                if text != '\n':
	                    t= text.split('-|-')
	                    email=t[2].split('@')
	                    temp['idnumber']=(t[0])
	                    temp['username']=(t[1])
	                    temp['emailfront']=(email[0])
	                    temp['emailprovider']=(email[1])
	                    temp['passwordhash']=(t[3].rstrip('='))
	                    temp['hint']=(t[4][:-4])
	                    temp=[t[0],t[1],email[0],email[1],t[3].rstrip('='),t[4][:-4]]
	            except IndexError:
	                #print('oops', text)
	                pass
	            else:
	                csvwriter.writerow(temp)


Next i wrote a small script to check for my email:

.. code:: python

	filename = 'converted.tsv'
	email = 'test'
	with open(filename,'r') as out_file:
	    for line in out_file:
	        if email in line:
	            print(line)

sample output:

.. code::

	12345990		name	gmail.com	 sKZcyHioxGNzioxG6CfCw	 dog


I like the pseudo code style of python :-). My password hint is dog and I can see my hash. Now my interest was peaked and I wanted to see what I can learn from this dataset. Here, I thought it is a good idea to learn a bit about pandas. I come from a Matlab environment for data wrangling and wanted to see what python has to offer.

Using `pandas <http://pandas.pydata.org/>`__ this is pretty straightforward. Since my computer doesn't have enough memory I used only the first 10000000 entries. I will probably next look into ways to work faster with that much data. Here is the iphyton notebook used:

.. code:: python

    import time
    filename= 'converted.tsv'
    import pandas as pd
    tic = time.clock()
    adobeDataFrames=pd.read_csv(filename, nrows=10000000, delimiter='\t', usecols=[2,3,4,5], names=['emailfront','emailprovider','passwordhash','hint'])
    toc = time.clock()
    print('[*] Time to read data: ',toc-tic)

.. parsed-literal::

    [*] Time to read data:  19.532169279833354
    

.. code:: python

    print('[*] rowcount: ', len(adobeDataFrames.index))

.. parsed-literal::

    [*] rowcount:  10000000
    
The top ten email providers:

.. code:: python

    print(adobeDataFrames.emailprovider.value_counts()[1:10])

.. parsed-literal::

    yahoo.com        1285637
    gmail.com         937736
    aol.com           315863
    msn.com           138874
    comcast.net       107684
    hotmail.co.uk      99280
    web.de             83145
    gmx.de             65824
    sbcglobal.net      64469
    dtype: int64
    
Top 20 password reminders, I was once a ``usual`` :-):

.. code:: python

    print('[*] hints:\n',adobeDataFrames.hint.value_counts()[1:20])

.. parsed-literal::

    [*] hints:
     name        53044
    ??          36036
    usual       34571
    ????        33202
    ???         25743
    me          25246
    same        23438
    cat         22477
    son         18065
    daughter    17497
    nickname    16957
    ?????       15753
    ??????      14079
    pet         13315
    work        12744
    normal      12544
    car         12042
    my name     11914
    love        11381
    dtype: int64
    
Top 20 email names, notice the absence of female names on the list: 

.. code:: python

    print('[*] front of email:\n',adobeDataFrames.emailfront.value_counts()[1:20])

.. parsed-literal::

    [*] front of email:
     webmaster     8236
    mail          7246
    admin         7216
    adobe         6471
    sales         4874
    john          4677
    chris         4522
    david         4388
    mike          4208
    mark          3568
    contact       3440
    paul          3408
    steve         3321
    macromedia    3194
    peter         2850
    michael       2828
    support       2818
    office        2802
    dave          2447
    dtype: int64
    
Now we come to an interesting part. Adobe used always the same algorithm to calculate the hash and did not salt the stored hashes. This results in having the same hash for the same passwords. Here we have a list of the top 20 hashes that are connected to the 20 most common passwords.

.. code:: python

    print('[*] passwordhashes:\n',adobeDataFrames.passwordhash.value_counts()[1:20])

.. parsed-literal::

    [*] passwordhashes:
     L8qbAD3jl3jioxG6CatHBw    37431
    j9p+HwtWWT86aMjgZFLzYg    23348
    j9p+HwtWWT/ioxG6CatHBw    14591
    5djv7ZCI2ws               13368
    7LqYzKVeq8I               10862
    dQi0asWPYvQ                9701
    ukxzEcXU6Pw                8474
    WqflwJFYW3+PszVFZo1Ggg     7904
    BB4e6X+b2xLioxG6CatHBw     6734
    diQ+ie23vAA                6726
    kCcUSCmonEA                6616
    e6MPXQ5G6a8                6311
    4V+mGczxDEA                5902
    PMDTbP0LZxu03SwrFUvYGA     5873
    xz6PIeGzr6g                4743
    hjAYsdUA4+k                4493
    5wEAInH22i4                4361
    rpkvF+oZzQvioxG6CatHBw     4245
    j9p+HwtWWT8/HeZN+3oiCQ     4142
    dtype: int64
    
To come full circle to the xkcd comic, we pull the most common hints for the ten most common passwords(hashes). It is left to the reader to solve these.

.. code:: python

    hash_list=hash_list=adobeDataFrames.passwordhash.value_counts()[1:10].index.tolist()
    for h in hash_list:
        print('[*] hash: ',h)
        print(adobeDataFrames.hint[adobeDataFrames['passwordhash']==h].value_counts()[1:10])

.. parsed-literal::

    [*] hash:  L8qbAD3jl3jioxG6CatHBw
    pw          282
    usual       261
    same        225
    easy        205
    hint        201
    word        172
    duh         141
    wordpass    141
    obvious     140
    dtype: int64
    [*] hash:  j9p+HwtWWT86aMjgZFLzYg
    123          496
    numbers      442
    1-9          286
    987654321    281
    123456       275
    number       180
    1            122
    19           109
    12345         96
    dtype: int64
    [*] hash:  j9p+HwtWWT/ioxG6CatHBw
    1-8         177
    123         136
    number      135
    numeros     113
    87654321     83
    1234         74
    123456       71
    18           57
    1            50
    dtype: int64
    [*] hash:  5djv7ZCI2ws
    ytrewq    149
    q         111
    qw         79
    asdfgh     67
    qwert      65
    qwe        63
    key        61
    qy         47
    123456     41
    dtype: int64
    [*] hash:  7LqYzKVeq8I
    222222     120
    111         63
    123456      62
    11          54
    one         50
    numbers     39
    61          37
    number      33
    6           31
    dtype: int64
    [*] hash:  dQi0asWPYvQ
    123         128
    numeros     123
    1-7         115
    7654321      90
    123456       81
    number       80
    1            50
    12           47
    12345678     47
    dtype: int64
    [*] hash:  ukxzEcXU6Pw
    Series([], dtype: int64)
    [*] hash:  WqflwJFYW3+PszVFZo1Ggg
    flash            13
    software          8
    macro             7
    company           6
    mac               5
    company name      4
    programa          4
    manufacturer      3
    software name     3
    dtype: int64
    [*] hash:  BB4e6X+b2xLioxG6CatHBw
    123           45
    usual         24
    adobe 123     21
    site          18
    software      18
    123adobe      18
    name          17
    company123    16
    Adobe         15
    dtype: int64
    
I will stop here. There is much more information in here. For instance the hash itself can tell you about the password length. An interesting article can be found on `naked security <https://nakedsecurity.sophos.com/2013/11/04/anatomy-of-a-password-disaster-adobes-giant-sized-cryptographic-blunder/>`__. I am very happy with my first pandas experience and definitivly use it again. 








