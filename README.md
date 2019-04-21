# Automated Player Station Logistics for X4

There currently is no real logistics functionality for stations. Each station is an independent entity that doesn't know or care that it's part of a larger network of faction owned stations. "Logistics" are limited to buy and sell orders, and having traders assigned to a station is really hit or miss. Said traders also won't prefer other faction owned stations over any other stations. Because they evaluate prices, it's all based on where the good can be got for the cheapest. That means there's no bigger logistical vision than an individual station and it's needs.

That's obviously a problem, particularly for the player, as it means the player (who will virtually always have a bigger picture in mind) can't actually achieve that vision, at least not without some really inefficient workarounds and micromanagement. The options currently for the player are either to restrict products from other factions and to manipulate prices such that only player tradeships will ever possibly be interested in them.

That's crap. The solutions mean excess production will never be sold to others. It also doesn't even guaruntee that resources will be aquired solely from other player owned stations when production is adequate, as NPC factions can always undercut the player on price. Not only that, but the entire system is based around buying and selling instead of transferring. Or in other words, the traders will always evaluate price even when price between player owned stations is irrelevant.

This mod aims to change the entire logistical system for the Player so that their stations will operate as a more effecient network.

Another goal is to minimize player interaction with the logistical system. The player hires station managers, and those managers should do the jobs they're (not) paid for. At most the player should only have to interact at a macro level. Assigning traders to stations, setting preferences for individual stations, etc...

It should largely be "set it and forget it" or maybe "set it and occaisionally monitor it". The main concern of the player should really be "Do I have enough production facilities placed strategically so outside purchases are unnecessary and do I have enough tradeships assigned to production stations (and/or miners for first tier products) to be able to fetch the needed wares?"


## System Overview:

At a high level, station managers should be the central point of the logistics network. They should figure out their resource needs, and then check with other stations to see if those needs can be met in-network before issuing buy orders for those that can't be. Trade ships assigned to the station should then be told to go transfer wares from in-network stations, and only if in-network production can't satisfy the demand should they seek to buy goods from other factions (after in-network available resources are exhausted, of course).

This simple change to how the system currently works would end up being much more effecient, and allow for excess wares to be sold on the open market bringing in additional revenue for the player in an intelligent manner.

### More in depth:

The system should effectively be one way: Those needing resources should be sending traders to where they can aquire said resources. Being bi-directional (sending traders to get goods from other player stations, as well as sending traders to take finished goods to other stations) is more complicated than I currently wish to tackle. This mod may well evolve into that in the future, but for now it's going to be upstream only.

Stations already calculate their demand based on their available storage, for each good produced (target stock level) and issue buy orders to fill to that level. It's currently very simple. What I instead aim to do is make this a bit more complex. They should retain their target stock level calculations, but instead of just issuing a buy order for resources, they should first check with other player stations and attempt to reserve wares for transfer. If there aren't enough wares for transfer to satisfy the demand, only the remaining amount should be done as buy orders (target stock - reserved from other stations = buy order). [FI: Criticality level where if a good is too far below target stock levels, station managers will just seek the closest available resource, player or not, and not really care about cost. Their goal is to keep production going.]

Additionally there should be a prioritization to determine which goods traders should be going out to get. This would simply be the good with the lowest percentage of stock (taking goods that are reserved for transfer into account). [FI: Allow the player to override this default priority by adjusting percentage of target stock levels.]

Stations that are targets to procure wares should likewise be reserving their products when notified by a downstream station they seek to transfer. This already exists in the game at the point an actual trade command is initiated (the seller knows someone is on the way to get goods, and holds those goods. The buiyer is aware goods are on the way, and subtracts that from their buy order). However I would seek to expand that so that the reservation happens before actual traders have orders. So long as a downstream station has at least 1 trader, it should be able to reserve wares at upstream stations. [FI: Upstream stations should be able to have a max reservable amount, with any excess from that going on the open market so that the player can guarantee every station can at least make some money directly, if desired. Other player stations will never actually buy these, they are for selling to others for income generation exclusively.]

If the downstream station has no traders, then no reservations can happen and the full resource demand will always be a buy order.

When attempting to reserve resources from player owned stations, it should start at the closest station and work out, up to its max range (Determined by manager level, currently for subordinate traders this appears to be level * 2, so a max of 10 sectors). [FI: Warehouse hubs. Oh yeah. A proper logistical network].

If there aren't enough player-produced wares to fill the need, at that point a buy order for the remainder will be issued, and any subordinate traders that aren't tied up fetching resources from player stations would then attempt to get them on the market. This should prioritize cheapest price from non-player stations within range (regardless of distance). [FI: Critically levels could change this to focus on distance instead.]

If the demand of a station is fully met, at that point subordinate traders should begin trying to make money. Don't want them just sitting around bored, afterall. How this is accomplished will depend on the output of the station they are subordinate to. 

They should first attempt to sell off excess products for their station (those not reserved by other stations and put up as sell orders). If there are no excess products, or there aren't enough to fill ~60-70% of their cargo hold, they should then be sent out as normal auto-traders (but with profits generated for the station itself, not going directly to the player's bank).

For miners, if the station has miners attached, it should basically never try to buy resources that are being mined. [FI: Criticality should play into this. If a ware is critically needed, it should then try to buy in addition to mining until no longer critical.].

All of this should culminate in an efficient logistical network with little waste, while remaining rather "hands off" for the player, so they aren't constantly trying to micromanage things. As new stations come online, there should remain little-to-no management by the player.

## Current Technical Issues

- The current system appears to be very ship focused. Being a subordinate appears to only modify the ship's ware list and range, other than that it looks like it's a normal trade behavior.

- I think the way it works is that when the player assigns a trader to a station, the lib.request.orders aiscript is run, and then the trader is assigned order.trade.routine aiscipt and just goes off to do its own thing based on that script. From what I can tell the station isn't actually managing anything.

lib.request.orders actually may serve as a good entrypoint for the ships. It looks like I could replace the current code which just sets up the trade routine to instead call a custom station trader script. That script could essentially just be a loop of "get thing to do from station, thing completed, restart". That would make sure the station manager doesn't have to poll its ships, and just essentially queues up (or determines at time a ship asks for something to do) orders.

- I would want to create "Transfer goods from player station", "trade to me", and "free trade" scripts (could likely leverage existing orders, and just modify them to conclude when all trades/transfers are done or even just use the same orders the player can give and wrap those up in a script).

The closer I can keep it to using vanilla code the better, as then I don't have to maintain as much and it should be at least somewhat resistent to changes.

- Need to look at all the order.trade.x aiscripts as I think some of those are what get called when the player manually specifies trades. Definitely need to find the reservation code.

- The station stuff looks like the real hard part. I can't even find what scripts are used to represent the station management stuff. I'm not at all sure where to even begin here. I need to find those scripts, and also whatever API stuff I have access to for reserving goods, storing variables, etc... Can I even create new variables for a station (mainly needed so I can keep track of resources to aquire from other player owned stuff vs buy orders).

- I did find the station mass traffic stuff. All of the above will need to account for mass traffic as well. In fact station managers should prefer mass traffic for wares transfers/buys as it frees up trade ships. The bigger question is can I set them to transfer instead of trade...

- Also need to figure out how scripts persist and what steps (if any) I need to do when a save is loaded.
