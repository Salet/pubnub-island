# Escape from PubNub Island
## Backstory
You awake, the lone survivor of a shipwreck on a mysterious island. After searching the island you discover a now abandoned communications system. If you can figure out how to work the system you just might be able to signal for help.

You've got access to the communications handbook but are missing certain configuration parameters. Luckily it seems the old system operators have hidden the necessary configuration through a series of puzzles.

To escape the island you need to crack the puzzles, correctly operate the communications system and broadcast your SOS message!

## Setup

I'm not sure if we can make getting the keys part of the game but if so, this has to happen first. 

I think instead however, we can just use a fixed keyset but randomize the channel names on start.
Each puzzle will be multiple choice and the answer will correspond to the relevant answer. This means that the puzzles will work for whichever channel names are generated

In easy mode, there is placeholder PubNub code - answer is enough
In hard mode, we pattern match(?) for the full correct code with answer

## Game Story / Levels
*Event* - Communication lockdown triggered

### 1 - Subscribe
* Need to ability to subscribe to messages to get the previous operators credentials.
* Solve this level's puzzle to determine which channel to listen on.

*Event* - Messages are now being received

### 2 - Stream Filter
* Once you start subscribing, you actually get lots of different credentials:
```
{
    meta: fake
    operator1: <fakeToken1>,
    operator2: <fakeToken2>,
    operator3: <fakeToken3>
}
---
{
    meta: real
    operator1: <authtoken1>,
    operator2: <authtoken2>,
    operator3: <authtoken3>
}
```

* Need to use a stream filter to determine the correct set.
* Riddle/puzzle for filter expression

*Event* - Only the real credentials are now being received

### 3 - PAM
* Only one operator is allowed to publish messages. The others will shutdown the system (lose game)
* Puzzle for this level is to determine which operator owns the correct auth token to use

*Event* - Publishing messages is now possible but you don't know the islands coordinates

### 4 - History
* Coordinates have been used in the past on channel X
* Need to use history on the correct channel to recall messages that contain coordinates.

*Event* - We now have permission to publish and know the correct payload to send

### 5 - Publish
* The puzzle for this level is to determine communication channel to publish to.
* The answer is the correct channel name

*Event* - Publish the correct coordinates with the correct authtoken to the correct channel to win.



## Possible Extra Levels

### Functions
* Need to use functions to interperet some clues, maybe a cipher solver?
* Riddle/puzzle gives the cipher key

### Presence
* Need to set the state of some system as online using presence.
* Puzzle will tell you what to set

### Objects
* Need to configure the system using objects channel meta.
* Clues will tell you what to set the channel name/description to.

## Puzzles

### Puzzle for subscribe

Question: If five cats can catch five mice in five minutes, how long will it take one cat to catch one mouse?

Answer: 5


### Puzzle for publish:

There are 3 crates in front of you. One contains only apples, one contains only oranges. The third crate contains a mixture of apples and oranges.

Someone has switched around the labels however so now none of the crates correctly match their associated label.

You can't look inside the crates but can remove one piece of fruit at a time.

What is the minimum number of fruits you need to inspect to be able to correctly identify all three crates?

This number is the channel that you need to use.

Answer 1

### Puzzle for stream filter
TODO

### Puzzle for PAM
TODO

### Puzzle for History
Tom, Larry and Richard built the communication system. The lead engineer broadcasts clues on his PubNub channel, which is named after him.

Which answer in the list is the correct answer to this question?

1. All of the below.
2. None of the below.
3. All of the above.
4. One of the above.
5. None of the above.
6. None of the above.

1 = Alpha
2 = Bravo
3 = Charlie
4 = Delta
5 = Echo
6 = Foxtrot

Echo

