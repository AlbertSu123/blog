---
title: Kinesis Labs
date: '2021-04-20'
tags: ['crypto', 'infra']
draft: false
summary: 'Problems encountered when running Kinesis Labs'
---

This one really sucks. It was a series of unfortunate mistakes that led to a multinational company repeatedly issuing DMCA takedown notices on our website.

Timeline of Events: 1. We create Kinesis Labs, a stableswap on Evmos. 2. I personally didn't know it at the time, but Kinesis is based off a character from Maplestory - named Kinesis. Maplestory is owned by Nexon Corporation 3. When we published the site, we bought the domain kinesislabs.co(this is fine, there are tons of products out there named Kinesis). The problem is that we used a picture off google images of the Maplestory Kinesis. 4. This image was only a temporary placeholder while we commissioned art(for 3k USD -\_-) from our artist. What they drew is pretty distinctive from the Maplestory Kinesis. 5. At the same time, we received two messages from someone in Nexon Corportaion in telegram and twitter asking us if we knowingly infriged copyright laws. Obviously, we ignore these messages, as we get tons of similar spam every day.
a. Side Note: I feel like even if we did respond to these messages, it wouldn't have done much good since it seems like the reporter already filed a complaint to Nexon and was trying to get a confession out of us. He wasn't trying to help us make sure our website was compliant.
b. Side Side Note: We looked at this guy's twitter profile. From what we've gathered, this is lieterally your average defi user who happened to follow us because they were interested in our project. They then decided to report our website to their employer/ 6. We ignore the message again, oops, and we receive notice of a DMCA on Netlify. If we don't respond, they will take down our site in 7 days. 7. The notice was extremely incomplete of course, the pictures they linked were no longer on our site, so we draft a reply. We also asked our friends for advice, most said we were fine, and one of our friends actually put us in contact with a law firm. We didn't immediately engage with the firm because it was expensive and we did not raise any money, so all legal fees would come from our own pockets. 8. We draft a reply, under my name, telling them the links they sent were invalid and no longer on our side. For some legal reason, drafting a reply opens you personally up to legal liability. Of course, this does nothing - apparently we filed the response in an incorrect format to AWS. 9. We migrate to Vercel in the meantime. We then get another DMCA on vercel - even though they linked the webpage from Netlify. 10. We draft up a response to vercel, the problem right now isn't just the legal issues but that fact that these hosting websites take own our site immediately upon receiving a DMCA notice, not after the dispute has been disputed.

The Bridge Wars
Cosmos is a horizontal scaling blockchain. Instead of creating rollups that handle computations, they allow users to create application specific blockchains. This brings us to Evmos, an application specific blockchain that runs the EVM. We mentally model Evmos as the Avalanche C-Chain of Cosmos. Evmos needs to connect to other non-Cosmos chains using bridges.

Evmos' current philosophy is to pick a single bridge and kingmake it. For some reason that we don't quite comprehend, they chose Nomad, a pretty small bridge that isn't well known and is pretty new. One major use case for Kinesis is to swap between stablecoins from different bridges. Thus, it would be beneficial for us if there was the concept of bridge wars, where multiple bridges try to become the dominant bridge by adding tons of liquidity from their bridge.

Thus, we've talked to many different bridges: Axelar, Celer, Gravity, Synapse, and Wormhole. It was pretty obvious that all of these bridges were more advanced and further along than Nomad, however, the Evmos team has decided to crown Nomad as the canonical bridge before the chain launched. This obviously pissed off the rest of the bridge teams, as they wanted a fair and organic competition.

Currently, we are floating the narrative that if these non-Nomad bridges heavily incentivize their pools, they will win the Evmos bridge wars and become the canonical bridge, but this will only work if Evmos becomes wildly successfully.

What Evmos Needs
Goals: - Establish a high market cap and FDV - Get lots of retail interest - Build a system that captures value into Evmos - Make it difficult to replicate Evmos due to network effects.

Problems: - No business development at all, just 1 person and 1 intern - Failed launch and a lack of narrative control - No large players have a vested interested in Evmos like (FTX:Solana), (Jump:Terra), (3AC:Avalanche)

Ideas: - Get a similar backer like Paradigm, GSR, Wintermute, Citadel, B2C2 - Establish a figurehead like Do Kwon or Vitalik. Average people will look to this person and go: this person is very smart and will pump my bags. - Create the Evmos project, similar to solidly on Fantom. Maybe a cross chain yield aggregator? Solidly airdropped 20 governance NFTs to the 20 highest TVL projects on Fantom, this generated huge hype and liquidity. - Pay off eth core devs to build on Evmos - Cevmos - Incentivize forks over native deployment, so that random Eth devs will see others making 1 mil launching forks on Evmos. - Evmos emoji - Waves play
