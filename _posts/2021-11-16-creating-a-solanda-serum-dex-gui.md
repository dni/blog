---
layout: post
title:  "creating a solana bases DEX (decentralized exchange) GUI with serum to earn referals trading (maybe) NFT's"
---

# introduction

how easy is it to create your own DEX and earn referal money? not so easy, as we are discovering now.

## the concept

### the idea
the idea was to create a server host a GUI for a DEX and earn money by getting people to trade on our dex and earn referals, basically a copy of sites like [radium](https://dex.raydium.io) and other projects of that kind, which is called "DEX"


### the execution
after some quick devops work, forking, installing and updating, we quickly discovered a problem with our project of choice [Serum Dex Ui Example](https://github.com/project-serum/serum-dex-ui). despite the documentation not mentioning it a bit, there is a subscription
for the datafeed you need to display correct chart data with tradingview on your DEX. the price is
1000€ per month to use their aggregated onchain data. bohoo, big bummer


### the conclusion

i spent 90% figuring out the complexity and the dependencies of the different services like "serum" und "bonfida" which is a paid subscription only to discover that without the data, you can pretty much only offer a orderbook directly from serum, but the question, will their service also become subscription only in the future? i'm not sure.

all that said, we still learned some valueable lessons and still here is some more stuff i gathered when trying to do this.


#### hosting

you can simply host it by creating a free github repository on github.com and fork the [serum-dex-ui](https://github.com/project-serum/serum-dex-ui) repository, customize it, and enable github pages.
you can clone it locally, modify your wallet addresses and rebuild and deploy your app with `yarn deploy` and everything on github pages will be updated


#### quick install development server
=====================================

```sh
git clone git@github.com:project-serum/serum-dex-ui.git
cd serum-dex-ui
yarn
yarn start
```

#### build and deploy to github pages
=====================================

```sh
yarn deploy
```

#### customize slogan, logos, colors, etc.

you modifiy the logo files in src/assets/logo.svg, public/
slogan: src/components/TopBar.tsx, LINE 145
for css you edit src/ant-custom.less

### links

* [Serum Dex Ui Example](https://github.com/project-serum/serum-dex-ui)
* [bonfida docs](https://docs.bonfida.com/)
* [bonfida docs tradingview](https://docs.bonfida.com/#trading-view)
* [serum docs](https://docs.projectserum.com/)
* [NEXTLEVELTRADING Discord](https://discord.gg/jYJ84tXMYV)

