<p align="center">
  <img 
    src="../assets/logo.svg" 
    width="200px"
    alt="Neo">
</p>

<p align="center" style="font-size: 48px;">
  <strong>Workshop 003: Smart Contracts 1 - Writing, Deploying, and Invoking</strong>
</p>

# Introduction
* <b>Duration:</b> 
	* 1 Hour
* <b>Prerequisites:</b> 
	* [002 Neo Development Environments](../2_development_environment/README.md)
* <b>Infrastructure Requirements:</b>
	* (Required) Whiteboard or Projector
	* (Required) High-Speed Internet Connection
	* (Required) Deployment of an Eventnet for workshop use
* <b>Instructor Prework:</b>
	* (Required) Workshop content review
	* (Required) Fill out the [worksheet](./Worksheet.md) for students
* <b>Student Prework:</b>
	* (Recommended) General Understanding of blockchain fundamentals
	* (Recommended) Familiarization with Amazon Web Services(or other CSP)
* <b>Workshop Materials List:</b>
	* None

## Outline
This workshop covers how to use the Neo development environment to develop basic smart contracts for the Neo platform. It will also cover the deployment process as well as how to invoke the contracts. This workshop will be branching and will support C#, python, and typescript contract development and their environments.


## Tools
* <i><b>Note:</b> All code examples are flagged with a language and tool for clarity.  Depending on your implementation path, you may be using cross language libraries (for example, writing a smart contract in python, but interfacing in javascript). </i>

## Introduction and Setup
In this course we will be walking students through smart contract development including writing, deploying, and invoking.  We will primarily be using NEP5 contracts as examples, but will cover additional contracts to provide feature coverage.  When developing a smart contract for Neo, its important to understand that the contract deployment workflow has an intermediary step where the contract code is compiled from its native language into a .avm file containing bytecode which the neoVM interprets.  


<p align="center">
  <img 
	src="../2_development_environment/assets/workflow.png" 
	alt="Neo">
</p>
	
	
## Writing a contract
In the NEO ecosystem, there are two triggers to consider when writing a smart contract.  These triggers are used as flow control. 
* <b>Application Triggers:</b>
Application triggers behave similarly to method calls.  These events occur when a user attempted to invoke a specific method exposed in the smart contract.
* <b>Verification Triggers:</b>
Verification triggers running when a verification occurs from a transaction.  For example, if an address sends NEO or GAS to the contract, the verification trigger is emitted.

### Events
Events are another important component to smart contracts because they significantly enhance the usability of the platform.  Because invokes are published using bytecode, its very difficult of applications to parse this information to detect events which may be relevant to their performance.  An event raises a flag to indicate that something occured.  In the NEP5 standard, we see a [transfer event](https://github.com/neo-project/proposals/blob/master/nep-5.mediawiki#events) which broadcasts that a token transfer event has taken place.  It also provides some additional useful information about the event.

### The Virtual Machine
The Neo platform runs smart contracts in a virtual machine.  This allows contracts to be written in any language for which there is a compiler.  Its very important to note the distinction between compiling from a lanaguage to byte code run by the VM and native support for the language.  If you are having issues with debugging a section of your code, its possible that a feature of your language may not yet be supported by the compiler.
Examples of the same method in multiple languages:
* C#: `Storage.Put(Storage.CurrentContext, Owner, pre_ico_cap);`
* Python: `Put(ctx, t_to, to_total)`
* Go: `storage.Put(ctx, to, totalAmountTo)`

These all result in similar bytecode once compiled.  You can refer to the complete list [<b>here</b>](https://docs.neo.org/developerguide/en/articles/neo_vm.html)



### Our Atomic NEP-5
In this workshop will make use of an atomic NEP-5 token.  The contract used may be referenced for learning purposes but you should always reference the latest templates for contracts which implement the most updated design patterns to ensure security.

* [<b>Python</b>](./assets/nep5.py)
* [<b>Go</b>](https://github.com/CityOfZion/neo-storm/tree/13b9dfa70af55896334ea02de3b5a05ec6dc4eee/examples/token)

<b>The Python Example:</b>
```python

"""
NEP5 Token
===================================

A simplified version of the original authored by: 
.. moduleauthor:: Thomas Saunders <tom@cityofzion.io>

..modifiedby:: Tyler Adams <tyler@coz.io>

"""

from boa.interop.Neo.Runtime import GetTrigger, CheckWitness
from boa.interop.Neo.Action import RegisterAction
from boa.interop.Neo.TriggerType import Application, Verification
from boa.interop.Neo.Storage import GetContext, Get, Put, Delete
from boa.builtins import concat

# -------------------------------------------
# TOKEN SETTINGS
# -------------------------------------------

OWNER = b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
# Script hash of the contract owner

# Name of the Token
TOKEN_NAME = 'NEP5 Standard'

# Symbol of the Token
TOKEN_SYMBOL = 'NEP5'

# Number of decimal places
TOKEN_DECIMALS = 8

# Total Supply of tokens in the system
TOKEN_TOTAL_SUPPLY = 10000000 * 100000000  # 10m total supply * 10^8 ( decimals)

ctx = GetContext()

# -------------------------------------------
# Events
# -------------------------------------------

OnTransfer = RegisterAction('transfer', 'addr_from', 'addr_to', 'amount')
OnApprove = RegisterAction('approve', 'addr_from', 'addr_to', 'amount')


def Main(operation, args):
    """
    This is the main entry point for the Smart Contract

    :param operation: the operation to be performed ( eg `balanceOf`, `transfer`, etc)
    :type operation: str
    :param args: a list of arguments ( which may be empty, but not absent )
    :type args: list
    :return: indicating the successful execution of the smart contract
    :rtype: bool
    """

    # The trigger determines whether this smart contract is being
    # run in 'verification' mode or 'application'

    trigger = GetTrigger()

    # 'Verification' mode is used when trying to spend assets ( eg NEO, Gas)
    # on behalf of this contract's address
    if trigger == Verification():

        # if the script that sent this is the owner
        # we allow the spend
        is_owner = CheckWitness(OWNER)

        if is_owner:

            return True

        return False

    # 'Application' mode is the main body of the smart contract
    elif trigger == Application():

        if operation == 'name':
            return TOKEN_NAME

        elif operation == 'decimals':
            return TOKEN_DECIMALS

        elif operation == 'symbol':
            return TOKEN_SYMBOL

        elif operation == 'totalSupply':
            return TOKEN_TOTAL_SUPPLY

        elif operation == 'balanceOf':
            if len(args) == 1:
                account = args[0]
                return do_balance_of(ctx, account)

        elif operation == 'transfer':
            if len(args) == 3:
                t_from = args[0]
                t_to = args[1]
                t_amount = args[2]
                return do_transfer(ctx, t_from, t_to, t_amount)
            else:
                return False

        return 'unknown operation'

    return False


def do_balance_of(ctx, account):
    """
    Method to return the current balance of an address

    :param account: the account address to retrieve the balance for
    :type account: bytearray

    :return: the current balance of an address
    :rtype: int

    """

    if len(account) != 20:
        return 0

    return Get(ctx, account)


def do_transfer(ctx, t_from, t_to, amount):
    """
    Method to transfer NEP5 tokens of a specified amount from one account to another

    :param t_from: the address to transfer from
    :type t_from: bytearray
    :param t_to: the address to transfer to
    :type t_to: bytearray
    :param amount: the amount of NEP5 tokens to transfer
    :type amount: int

    :return: whether the transfer was successful
    :rtype: bool

    """

    if amount <= 0:
        return False

    if len(t_from) != 20:
        return False

    if len(t_to) != 20:
        return False

    if CheckWitness(t_from):

        if t_from == t_to:
            print("transfer to self!")
            return True

        from_val = Get(ctx, t_from)

        if from_val < amount:
            print("insufficient funds")
            return False

        if from_val == amount:
            Delete(ctx, t_from)

        else:
            difference = from_val - amount
            Put(ctx, t_from, difference)

        to_value = Get(ctx, t_to)

        to_total = to_value + amount

        Put(ctx, t_to, to_total)

        OnTransfer(t_from, t_to, amount)

        return True
    else:
        print("from address is not the tx sender")

    return False
	
```



<b>To Build into a .avm:</b>
* <b>Python:</b> In the neo-python prompt

   ```sc build {{path to smart contract}}```
   
* <b>Go:</b> From shell

    ```neo-storm compile -i path/to/file.go```


* <b>C#:</b> (Built from compiler)
    
### Other Examples:

#### <b>C#</b>:
* [Vanilla NEP-5 template](https://github.com/neo-project/examples-csharp/blob/master/ICO_Template)
* [Moonlight NEP-5 template](https://github.com/Moonlight-io/moonlight-ico-template)
* [Neo Docs Samples](https://docs.neo.org/en-us/sc/tutorial/HelloWorld.html)
	
#### <b>Python</b>:
* [NEX NEP-5 Template](https://github.com/nash-io/neo-ico-template)

#### <b>Go</b>:
* [neo-storm examples](https://github.com/CityOfZion/neo-storm/tree/13b9dfa70af55896334ea02de3b5a05ec6dc4eee/examples)


<b>For a list of supported methods:</b>
* [<b>Python:</b>](https://neo-boa.readthedocs.io/en/latest/index.html)
* [<b>Go:</b>](https://github.com/CityOfZion/neo-storm/tree/13b9dfa70af55896334ea02de3b5a05ec6dc4eee/examples)
* [<b>C#</b>](https://docs.neo.org/en-us/sc/reference/api.html)

## Local Invokes
Now that we have a compiled contract, lets run some invokes on it.  In NEO, we <b>HIGHLY</b> recommend leveraging testing in a local VM before deploying to the blockchain (even a privatenet) for efficiency.  Alternatively, you can deploy the contract to the privatenet and run system-level unit testing.

* <b>Python:</b> In the neo-python prompt
	```sc build run {{path to smart contract}} True False False  0710 05 name```
* <b>Go:</b>
	
* <b>C#:</b>

## Deploying a Contract
* Overview of deployment costs
* Deployment example on event-net

## Invoking a Contract
* Invocation costs
* Example invokation using neo-gui
* Example invocation using neon.js
* Example invocation using neo-python

### Local Invokes v. Published Invokes
* Differences in how each works
* How to leverage the difference in your architecture

### Notification Events