<p align="center">
  <img src="https://i.imgur.com/78yK1sr.png" />
  <br>
  <i>A probabilistic player rating system for Counter Strike: Global Offensive, powered by machine learning</i>
</p>

---

[![Latest release](https://img.shields.io/github/v/release/Phil-Holland/csgo-impact-rating?label=release&sort=semver)](https://github.com/Phil-Holland/csgo-impact-rating/releases)
[![Build Status](https://travis-ci.org/Phil-Holland/csgo-impact-rating.svg?branch=master)](https://travis-ci.org/Phil-Holland/csgo-impact-rating)
[![Go Report Card](https://goreportcard.com/badge/github.com/Phil-Holland/csgo-impact-rating)](https://goreportcard.com/report/github.com/Phil-Holland/csgo-impact-rating)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

<p align="center">
  <img src="https://i.imgur.com/EBbyDLv.png" />
  <a href='#how-it-works'>How it Works</a> • <a href='#prediction-model'>Prediction Model</a> • <a href='#download'>Download</a> • <a href='#usage'>Usage</a> • <a href='#built-with'>Built With</a> • <a href='#donate'>Donate</a> • <a href='#license'>License</a>
</p>

## How it Works

Impact Rating uses a machine learning model trained on a large amount of historical data to **predict the probable winner** of a given CS:GO round, based on the current state of the round. A **player's Impact Rating** is then calculated as the amount by which their actions shift the likelihood of their team winning the round. Therefore, **players are rewarded purely for making plays that improve their team's chance of winning the current round**. Conversely, negative Impact Rating is given when a player's actions reduce their team's chance of winning the round.

### Example

Two simplified examples of in-game scenarios are shown in the diagram below. Both describe a single CT player getting a triple kill, but they only receive impact rating in one scenario. In the first, the CTs are left in a 5v2 situation in which they are highly favoured to subsequently win the round. In the second, the remaining CTs are forfeiting the round, not attempting a late retake. A triple kill at this point **does not alter the CT team's chance of winning the round**.

![](https://i.imgur.com/vEMUxnD.png)

### Case Study

An in-depth case study analysis of an individual match between Mousesports and Faze can be found here: [case study](https://nbviewer.jupyter.org/github/Phil-Holland/csgo-impact-rating/blob/master/notebooks/case_study.ipynb). This analysis breaks down the events of a round, showing how model prediction values change accordingly, and the resulting effect on player Impact Ratings.

### Calculating Impact Rating

Internally, the state of a round at any given time is captured by the following features:

- CT players alive
- T players alive
- Mean health of CT players
- Mean health of T players
- Mean value of CT equipment
- Mean value of T equipment
- Whether the bomb has been defused  - *this is required to reward players for winning a round by defusing*
- Round time elapsed (in seconds)
- Bomb time elapsed (in seconds) - *this is zero before the bomb is planted*

Inputs for each of these features are passed to the machine learning model, which returns a single floating-point value between `0.0` and `1.0` as the round winner prediction. The value can be directly interpreted as the probability that the round will be won by the T side. 

> For example, a returned value of `0.34` represents a predicted **34% chance** of a **T side** round win, and a **66% chance** of a **CT side** round win.

This concept is applied to **every change in a round's state** from the end of freezetime to the moment the round is won. Players that contributed to changing the round outcome prediction are rewarded with a **appropriate share of the percentage change** in their team's favour - the **sum of these values** over a particular round is their Impact Rating for that round. The following actions are considered when rewarding players with their share:

- Doing damage
- Trade damage - *if an opponent takes damage very soon after they themself have damaged the player in question*
- Flash assist damage - *if someone takes damage whilst blinded by a flashbang thrown by the player in question*
- Defusing the bomb
- Being "defused on" - *if a T side player is alive when the bomb is defused*
- Sustaining damage (being hurt)

> It is important to note that some actions may result in negative Impact Rating being "rewarded". For example, sustaining a large amount of damage may shift the round outcome prediction in the favour of your opponents, and therefore you should be punished with the corresponding negative Impact Rating. This also applies to team damage, or if a teammate takes damage after you teamflash them.

## Prediction Model

Whilst the concept behind Impact Rating can in theory be implemented using any binary classification model, the code here has been written to target the [LightGBM framework](https://github.com/Microsoft/LightGBM). This is a framework used for gradient boosting decision trees (GBDT), and has been [shown to perform very well](https://github.com/microsoft/LightGBM/blob/master/docs/Experiments.rst) in binary classification problems. It has also been chosen for its lightweight nature, and ease of installation.

Model analysis and instructions for how to train a new model can be found here: [model training & analysis](lightgbm/README.md).

## Download

The latest Impact Rating distribution for your system can be downloaded from this project's release page here: 

<p align="center">
  <a href="https://github.com/Phil-Holland/csgo-impact-rating/releases/latest">
    <b>Download Latest Version</b>
  </a>
</p>

Extract the executable and the `LightGBM_model.txt` file to a directory of your choosing - add this directory to the system path to access the executable from any location.

## Usage

CS:GO Impact Rating is distributed as a command line tool - it can be invoked only through a command line such as the Windows command prompt (cmd.exe) or a Linux shell.

```
Usage: csgo-impact-rating [OPTION]... [DEMO_FILE (.dem)]

Tags DEMO_FILE, creating a '.tagged.json' file in the same directory, which is
subsequently evaluated, producing an Impact Rating report which is written to
the console and a '.rating.json' file.

  -f, --force                Force the input demo file to be tagged, even if a
                             .tagged.json file already exists.
  -s, --eval-skip            Skip the evaluation process, only tag the input
                             demo file.
  -m, --eval-model string    The path to the LightGBM_model.txt file to use for
                             evaluation. If omitted, the application looks for
                             a file named "LightGBM_model.txt" in the same
                             directory as the executable.
  -v, --eval-verbosity int   Evaluation console verbosity level:
                              0 = do not print a report
                              1 = print only overall rating
                              2 = print overall & per-round ratings (default 2)
```

For general usage, the command line flags can be ignored. For example, the following command will **produce player ratings** for a demo file named `example.dem` in the working directory:

```sh
csgo-impact-rating example.dem
```

A full per-player Impact Rating report will be shown in the console output.

## Built With

- [demoinfocs-golang](https://github.com/markus-wa/demoinfocs-golang) - used to parse CS:GO demo files
- [leaves](https://github.com/dmitryikh/leaves) - used to process LightGBM models internally
- [pflag](https://github.com/spf13/pflag) - used to build the command line interface
- [pb (v3)](https://github.com/cheggaaa/pb) - used for progress visualisation
- [LightGBM](https://github.com/Microsoft/LightGBM) - used for model training/round outcome prediction

## Donate

CS:GO Impact Rating is completely free to use, so any donations are incredibly appreciated! 

Buy me a coffee with the following link:

[![Buy Me A Coffee](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/PhilHolland)