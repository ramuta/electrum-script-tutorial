<!---
Electrum csv2paytomany script
Copyright (C) 2014  Matej Ramuta

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License along
with this program; if not, write to the Free Software Foundation, Inc.,
51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
-->

How to write scripts for the Electrum wallet
=======

What is the Electrum bitcoin wallet?
-----------
Electrum wallet is a [bitcoin wallet](https://bitcoin.org/en/choose-your-wallet) that helps you receive, 
store and send bitcoins. It’s a [SPV](https://en.bitcoin.it/wiki/Scalability#Simplified_payment_verification) 
client which means it doesn’t download the whole Blockchain and thus saves you a significant space on your disk.

You’ll need an Electrum wallet and some small amount of bitcoins in it for this tutorial. Download it here: 
[http://electrum.org](https://electrum.org/).

Electrum console
-----------
Electrum has a Python console for everyone who prefers the command line over the graphical interface.

Go to Electrum console and try some commands:

The first one is “the usual suspect”: *help()*


    >>help()
    List of commands: contacts, create, createmultisig, createrawtransaction, daemon, decoderawtransaction, deseed, 
    dumpprivkey, dumpprivkeys, freeze, getaddressbalance, getaddresshistory, getaddressunspent, getbalance, getconfig, 
    getmpk, getproof, getpubkeys, getrawtransaction, getseed, getservers, getutxoaddress, getversion, help, history, 
    importprivkey, listaddresses, listunspent, mksendmanytx, mktx, password, payto, paytomany, restore, sendrawtransaction, 
    setconfig, setlabel, signmessage, signrawtransaction, unfreeze, validateaddress, verifymessage


*help()* command gives you a list of all possible commands that you can execute in the Electrum console. Let’s try another 
one, *getbalance()*

    >> getbalance()
    {
    "confirmed": "0.0076"
    }

*getbalance()* lets you know how much bitcoins you have in your wallet - how many of them are confirmed and how many not.

Example: Electrum script that sends bitcoins
-----------
Let’s try something more complex than just checking your wallet’s balance. Let’s say you want to send different amounts 
of bitcoins to different addresses. First create a CSV file that contains addresses and amounts of bitcoins, like that:

    address,amount
    1CM6v8N9ZST5koc8g7W2So6hKyADQYvi8B,0.001
    13F8ZZ11BWCAWkhvBsJJyUN3X9qz5Mef3Z,0.002
    16SjLAi2yTDShMoSHHA1RihsuVQZn3j1jY,0.003

CSV means “comma separated values” so addresses and amounts are separated with comma. Those of you who use a comma for 
a decimal mark please use a dot instead (I’m looking at you fellow Europeans) :) Save that CSV file under the name 
*sendlist.csv*.

Next create a python script with a name *electrum-script.py*. First we need to open our CSV file:

    import csv
    input_file = csv.DictReader(open("/full/path/to/your/sendlist.csv"))

Make sure that you include the FULL path to your CSV file and not just call *sendlist.csv*, even if CSV is in the same 
folder as your python script. This will be important later when we run the script in the Electrum console.

Next we have to store addresses and amounts in the python list as [tuples](https://docs.python.org/2/tutorial/datastructures.html#tuples-and-sequences):

    outputs = []
    
    for row in input_file:
        tup = (row["address"], float(row["amount"]))
        outputs.append(tup)
    
    print outputs

With print outputs we can make sure we have the right values stored in our list.

If we don’t it’s good to have a safeguard to cancel the process before the bitcoin transaction is made. So let’s ask a 
user for approval:

    approval = raw_input("Do you approve the transaction? (Y/N)")

If our user approves the transaction run the Electrum command that sends bitcoins to multiple addresses, called 
*paytomany()*

    if approval == "Y" or approval == "y":
        transaction = paytomany(outputs, 0.0001)
        print transaction
    else:
        print("Transaction was not executed.")

*paytomany()* command takes two arguments: a list of addresses and amounts (outputs) and the mining fee (in this case 
0.0001). Mining fee is optional but if you want your transaction to be processed quickly it is good to add it. 

So now we have everything ready to run the script! Go to Electrum console and run it with the *execfile()* command:

    execfile("/full/path/to/electrum-script.py")

Again, it’s important to include the full path to your script in order to run it. When you call the *paytomany()* command 
you’ll be asked to enter your password. After the successful transaction you get the transaction ID which you can check 
at [blockchain.info](https://blockchain.info/).

And that’s it! You can see the whole python script here: []()

You can also see a list of all the Electrum commands and the attributes they take here: 
[https://github.com/spesmilo/electrum/blob/master/lib/commands.py](https://github.com/spesmilo/electrum/blob/master/lib/commands.py )
