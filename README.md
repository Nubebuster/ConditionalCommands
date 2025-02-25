[![Build Status](https://travis-ci.org/konsolas/ConditionalCommands.svg?branch=master)](https://travis-ci.org/konsolas/ConditionalCommands)
# ConditionalCommands
Only execute a command if a condition is met.

ConditionalCommands is intended to be used when plugins have automatic commands that should only be executed if certain conditions are met. It is able to execute multiple commands, with customisable delays on each command.

## Usage
```
/cc <player> unless <condition> do <command>
/cc <player> if <condition> do <command>
/cc help
/cc reload
/cc cooldown <arbitrarykey>
```
Alias: `/ccmd`

### Conditions
Grammar of conditions:
```
<expression>::=<term>{<or><term>}
<term>::=<factor>{<and><factor>}
<factor>::=<comparison>|<not><factor>|(<expression>)
<comparison>::=<constant><comparator><constant>
<constant>::=floating point number or integer
<and>::='&'
<or>::='|'
<not>::='!'
<comparator>::='>'|'='|'<'
```
As shown above, only numbers can be compared, and placeholders can only consist of numbers. In case of multiple comparison operators in a group, i.e. 3>=<2, only the first operator will be used. Comparisons cannot include spaces. Inequality may be checked with !(value=value).

Examples:
```
/cc konsolas unless -ping->200 do kick konsolas
/cc konsolas if (-ping-<300&-ping->100)&-tps->15.0 do msg konsolas Your ping is between 300 and 100, and the TPS is greater than 15.
```

### Placeholders
Placeholders are delimited by '-'. Since they're applied with a replace, errors will probably be detected during parsing if they are typed incorrectly.

 - ```ping``` - The latency of the tested player.
 - ```tps``` - Server TPS average over the last 2 seconds
 - ```time_online``` - Player's online time in milliseconds
 - ```uptime``` - Server uptime in ticks
 - ```player_count``` - Number of players on the server
 - ```perm:<permission>``` - 1.0 if the player has the permission, 0.0 otherwise. e.g. ```-perm:essentials.home-```
 - ```perm_count:<permission>``` - Number of players online who have the permission, 0.0 otherwise. e.g. ```-perm_count:essentials.home-```
 - ```aacvl:<check>``` - AAC 1-4 violation level of the given check (internal name). e.g. ```-aacvl:speed-```
 - ```chance:<percentage>%``` - Will be 1.0 percentage% of the time. e.g. ```-chance:34.5%-```
 - ```cooldown:<arbitrarykey>``` - Returns time in seconds since the last execution of the ```/cc cooldown <arbitrarykey>``` command. Returns ```43200``` if ```/cc cooldown``` has never been executed for the given ```<arbitrarykey>```. Never returns a value < 0.
   e.g. ```/cc if -cooldown:some_key->59 do /0/ cc cooldown some_key /0/ broadcast sent up to once every 60 seconds```
   Maximum supported cooldown is 12 Hours aka 43200 seconds. Cooldowns are not saved to file, so they reset on server reload/restart.
   Use for example the player placeholder inside aac's configuration as part of the arbitrarykey.

#### With [spark](https://spark.lucko.me/]) installed
- ```tps:<window>```\
**Windows:** 5s, 10s, 1m, 5m, 15m
- ```mspt:<function[<arg>, <arg2*>]>```\
**Functions:** min[window], max[window], mean[window], median[window], percentile[percentile, window]\
**Windows:** 10s, 1m\
**Returns:** the tick time in milliseconds\
**Examples**:
  - mspt:min[10s]
  - mspt:percentile[95, 1m]

- ```cpu:<function[<window>]>```\
  **Functions:** sys[window], proc[window]\
  **Windows:** 10s, 1m, 15m\
  **Returns:** on a scale from 0.0 to 1.0\
  **Examples**:
    - mspt:sys[10s]
    - mspt:proc[1m]

**Note:** ```tps``` without parameters will stop working if spark is installed!

### Multi command / delayed commands
In the 'do' clause of the statement, multiple commands can be executed at once, and selected commands can be delayed if desired. The command delimiter is ```/<delay>/```, where the integer between ```/``` and ```/``` denotes the delay before the command should be executed in ticks. Here are some examples:

```
/cc konsolas if -aacvl:heuristics->0 do /1200/ ban konsolas 1 minute delayed ban for killaura
/cc konsolas if -perm:some.permission-=1 do /0/ broadcast konsolas has some.permission! /0/ broadcast second broadcast! /20/ broadcast 1 second later!
```

### Developer mode
By default, ConditionalCommands will suppress details behind parsing errors, etc. to be more user friendly. It will also swallow exceptions generated by executing a command. This behaviour can be changed by changing the ```dev``` option in ```plugins/ConditionalCommands/config.yml``` to true.
