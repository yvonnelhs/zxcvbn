```
_________________________________________________/\/\___________________
_/\/\/\/\/\__/\/\__/\/\____/\/\/\/\__/\/\__/\/\__/\/\________/\/\/\/\___
_____/\/\______/\/\/\____/\/\________/\/\__/\/\__/\/\/\/\____/\/\__/\/\_
___/\/\________/\/\/\____/\/\__________/\/\/\____/\/\__/\/\__/\/\__/\/\_
_/\/\/\/\/\__/\/\__/\/\____/\/\/\/\______/\______/\/\/\/\____/\/\__/\/\_
________________________________________________________________________
```

This is a Typescript port of Dropbox's [zxcvbn](https://github.com/dropbox/zxcvbn) library.

[![Build Status](https://travis-ci.org/trichards57/zxcvbn.svg?branch=master)](https://travis-ci.org/trichards57/zxcvbn)
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)
[![Coverage Status](https://coveralls.io/repos/github/trichards57/zxcvbn/badge.svg?branch=master)](https://coveralls.io/github/trichards57/zxcvbn?branch=master)
[![npm version](https://badge.fury.io/js/zxcvbn-typescript.svg)](https://badge.fury.io/js/zxcvbn-typescript)

`zxcvbn-typescript` is a password strength estimator inspired by password crackers. Through pattern matching and conservative estimation, it recognizes and weighs 30k common passwords, common names and surnames according to US census data, popular English words from Wikipedia and US television and movies, and other common patterns like dates, repeats (`aaa`), sequences (`abcd`), keyboard patterns (`qwertyuiop`), and l33t speak.

Consider using zxcvbn-typescript as an algorithmic alternative to password composition policy — it is more secure, flexible, and usable when sites require a minimal complexity score in place of annoying rules like "passwords must contain three of {lower, upper, numbers, symbols}".

* __More secure__: policies often fail both ways, allowing weak passwords (`P@ssword1`) and disallowing strong passwords.
* __More flexible__: zxcvbn allows many password styles to flourish so long as it detects sufficient complexity — passphrases are rated highly given enough uncommon words, keyboard patterns are ranked based on length and number of turns, and capitalization adds more complexity when it's unpredictaBle.
* __More usable__: zxcvbn is designed to power simple, rule-free interfaces that give instant feedback. In addition to strength estimation, zxcvbn includes minimal, targeted verbal feedback that can help guide users towards less guessable passwords.

For further detail and motivation, please refer to the USENIX Security '16 [paper and presentation](https://www.usenix.org/conference/usenixsecurity16/technical-sessions/presentation/wheeler).

# Installation

``` shell
npm install zxcvbn
```

``` shell
yarn install zxcvbn
````

## Browserify / Webpack

Bundling zxcvbn-typescript into your website using tools like browserify and webpack is unwise.  It is large, you likely only need it on a couple of pages
(registration, password reset), and you likely don't need it immediately.  If you are using browserify, consider using 
[dynamic imports](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-4.html#dynamic-import-expressions) and configuring your bundler
to allow lazy loading.

# Usage

[try zxcvbn interactively](https://lowe.github.io/tryzxcvbn/) to see these docs in action.

``` typescript
zxcvbn(password, user_inputs=[])
```

`zxcvbn()` takes one required argument, a password, and returns a result object with several properties:

``` typescript
{
  guesses : number, // estimated guesses needed to crack password
  guesses_log10 : number, // order of magnitude of guesses

  crack_times_seconds : // dictionary of back-of-the-envelope crack time estimations, in seconds, based on a few scenarios:
  {
    // online attack on a service that ratelimits password auth attempts.
    online_throttling_100_per_hour : number,

    // online attack on a service that doesn't ratelimit, or where an attacker has outsmarted ratelimiting.
    online_no_throttling_10_per_second : number,

    // offline attack. assumes multiple attackers,  proper user-unique salting, and a slow hash function
    // w/ moderate work factor, such as bcrypt, scrypt, PBKDF2.
    offline_slow_hashing_1e4_per_second : number,

    // offline attack with user-unique salting but a fast hash function like SHA-1, SHA-256 or MD5. A wide range of
    // reasonable numbers anywhere from one billion - one trillion  guesses per second, depending on number of cores 
    // and machines. ballparking at 10B/sec.
    offline_fast_hashing_1e10_per_second : number,
  },
  crack_times_display, // same keys as result.crack_times_seconds, with friendlier display string values: "less than a second", "3 hours", "centuries", etc.
  score : number,      // Integer from 0-4 (useful for implementing a strength bar)
                       // 0 - too guessable: risky password. (guesses < 10^3)
                       // 1 - very guessable: protection from throttled online attacks. (guesses < 10^6)
                       // 2 - somewhat guessable: protection from unthrottled online attacks. (guesses < 10^8)
                       // 3 - safely unguessable: moderate protection from offline slow-hash scenario. (guesses < 10^10)
                       // 4 - very unguessable: strong protection from offline slow-hash scenario. (guesses >= 10^10)
  feedback : // verbal feedback to help choose better passwords. set when score <= 2.
  {
    warning : string, // explains what's wrong, eg. 'this is a top-10 common password'.  Not always set -- sometimes an empty string
    suggestions : string[],  // a possibly-empty list of suggestions to help choose a less guessable password. eg. 'Add another word or two'
  },
  sequence : [], // the list of patterns that zxcvbn-typescript based the guess calculation on.
  calc_time : number // how long it took zxcvbn-typescript to calculate an answer in milliseconds.
}
````

The optional `user_inputs` argument is an array of strings that zxcvbn-typescript will treat as an extra dictionary. This can be whatever list of strings you like, but is meant for user inputs from other fields of the form, like name and email. That way a password that includes a user's personal information can be heavily penalized. This list is also good for site-specific vocabulary — Acme Brick Co. might want to include ['acme', 'brick', 'acmebrick', etc].

# <a name="perf"></a>Performance

## runtime latency

zxcvbn operates below human perception of delay for most input: ~5-20ms for ~25 char passwords on modern browsers/CPUs, ~100ms for passwords around 100 characters. To bound runtime latency for really long passwords, consider sending `zxcvbn()` only the first 100 characters or so of user input.

# Development

Bug reports and pull requests welcome!

zxcvbn-typescript is built with TypeScript, browserify, and uglify-js. Source lives in `src`, which gets compiled, bundled and minified into `dist/zxcvbn.js`.

Two source files, `adjacency_graphs.ts` and `frequency_lists.ts`, are generated by python scripts in `data-scripts` that read raw data from the `data` directory.

For node developers, in addition to `dist`, the zxcvbn `npm` module includes a `lib` directory (hidden from git) that includes one compiled `.js` and `.js.map` file for every `.ts` in `src`.  The type definitions produced by Typescript are there as well.

# Acknowledgments

Dan Wheeler for the initial zxcvbn project.
[Dropbox](https://dropbox.com) for supporting open source!

Mark Burnett for releasing his 10M password corpus and for his 2005 book, [Perfect Passwords: Selection, Protection, Authentication](http://www.amazon.com/Perfect-Passwords-Selection-Protection-Authentication/dp/1597490415).

Wiktionary contributors for building a [frequency list of English words](http://en.wiktionary.org/wiki/Wiktionary:Frequency_lists) as used in television and movies.

Researchers at Concordia University for [studying password estimation rigorously](http://www.concordia.ca/cunews/main/stories/2015/03/25/does-your-password-pass-muster.html) and recommending zxcvbn.

And [xkcd](https://xkcd.com/936/) for the inspiration :+1::horse::battery::heart:
