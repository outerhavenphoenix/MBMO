Description
 
    OHP's Mother Base Management Overhaul (MBMO) for MGSV. Rebalances Mother Base/FOB costs & development times.

About

    I really just want redo the whole base system. But I had to settle doing it this way for now.

Reminder

    Values that have 'e' are just LUA's way of expressing large numbers.
    For example, 1e3 = 1000 (don't blame me for values that don't have it, blame Konami :) )
    I try to stick to intervals of 3 as it's easier to read ( example 1e3 = 1000, 10e3 = 10000, 1e6 = 1000000).

TO DO

    // Up GMP rates of minor and rare metals
    // Decrease auto extract outputs
    // Increase proccessing rates

    // Workaround to fulton'ing shipping containers (not being able to extract containers until mission 10 is *extremely* limiting)
    // Adjust dispatch times to conside NEAR rank&period assumptions.
    // Add dispatch missions that include further ways to get materials.
        
    // Overhaul research requirements. Once I figure out how to deobfuscate Konami's TERRIBLE research table (don't EVEN get me started on how these LUA scripts are written...)
    // Investigate if Konami designed development system to have additional requirements for resources. (It would seem as if they don't know how to use embedded tables or meta tables :( )
    // Seriouly, Konami, kids that develop *ROBLOX* games can code LUA better than you.
    
    // Figure out what proccessingCount does
        TppMotherBaseManagement.SetResourceSvars{
            resource = "CommonMetal",
            usableCount = 100,
            processingCount = 20000,
            got=true,
            isNew=false
        }

About the logic

    // The entire idea is, this is how long it'd take you to collect this many resources in a period of time (based on rank/progress).
    // Time periods (by rank, this tries to consider you are as good as where your rank functions are at.)
        // F: 120 mins - 1 period
        // E: 60 mins - 2 periods
        // D: 40 mins - 3 periods
        // C: 30 mins - 4 periods
        // B: 24 mins - 5 periods
        // A: 20 mins - 6 periods
        // S: 15 mins - 8 periods
    // Target materials per period
        // TOTAL: 5050
        // Fuel: 2.5k (50%)
        // Common Metal: 1.25k (25%)
        // Biotic: 1k (20%)
        // Minor Metal: 200 (3%)
        // Precious Metal: 100 (2%)
    // 184k GMP ceiling per period (Essentially, this is the max you can sell for your mining+process output, at each period, for this amount)
        // Fuel: 3 GMP per unit (3%/5k)
        // Common: 10 GMP per unit (11%/12.5k)
        // Biotic: 30 GMP per unit (16%/30k) 
        // Minor Metal: 220 GMP per unit (23%/44k)
        // Precious Metal: 780 GMP per unit (45%/84k)

    // More materials requirements (common, fuel, and biotic TBD others)
        // BASE UPGRADES
            // Each upgrade should take about 2 cycles (~4 hours) to achieve each (or until you build FOBs to add to proccessing rates).
            // Grade 4 has +5 individual limits for mother base only (I'm considering adding more personnel space to other grads, due to.. cough... COVID-19/culling/plague in the game)
            // Costs: 
                // Grade 1: 1 period
                // Grade 2: 4 periods
                // Grade 3: 5 periods
                // Grade 4: 6 periods
            // Development times: (this is just done based on average playthrough, also so it doesn't feel "cheaty". You are likely to be on mission 10-30 by grade 3.)
                // Grade 1: Instant
                // Grade 2: 30 minutes
                // Grade 3: 40 minutes
                // Grade 4: 10 minutes (Likely getting close to end of game.)
        // FOB Modifier !!! TBD, I don't play online so I haven't throughly tested this yet.
            // +15% on FOB base upgrades (each FOB will add ~%50 of total mother base proccesing output, AND whatever mining rates are set for each FOB).


Automagic mining interval

    // In order to payout, it requires mission progress. It will only pay out once until the game is progressed or registers a checkpoint.
    // Going by how Konami probably designed the "anti-AFK" system. I'd recommend keeping the interval within a time of a checkpoint. (between 5 and ~25 minutes)
    // By default this is 1, so you really can't get any better than that.
    TppMotherBaseManagement.RegisterResourceBaseExtractingTimeMinute {
        timeMinute = 120 --// Time in minutes
    }

Automagic mining outputs by base and rank

    // This table effects ALL bases. These base amounts are redistributed by the engine.
    // If my count is 1000, fuel rate being 30, and minor rate being 20. There is a chance ~5% (somewhere in 5%) is added/removed from each value. So, the output may be fuel 295 and minor comes up to 205.
    // This is done to vary the outputs so it's not always the same per material.
    TppMotherBaseManagement.RegisterAutoResourceParam {
        oceanAreaId = 1,        --// AreaID of the FOD (1 = Mother Base)
        sectionFuncRank = "D",  --// Rank of base development function
        count = 7e3,            --// Total amount of resources produced (This will always be the total of the output)
        commonMetalRate = 29,   --// Weight produced (Rough percentage of total)
        minorMetalRate = 10,    --// Weight produced (Rough percentage of total)
        preciousMetalRate = 2,  --// Weight produced (Rough percentage of total)
        fuelResourceRate = 30,  --// Weight produced (Rough percentage of total)
        bioticResourceRate = 29 --// Weight produced (Rough percentage of total)
    }

Processing time (of raw materials) PER base development rank function

    // In order to payout, it requires mission progress. It will only pay out once until the game is progressed or registers a checkpoint.
    // Going by how Konami probably designed the "anti-AFK" system. I'd recommend keeping the interval within checkpoint rhythm (I like to think it's ~25 min average.)
    // ALSO, I'd recommend keeping these the same (in the rank.) Otherwise, you might find yourself getting notifications at different checkpoints (or missing outputs.)
    TppMotherBaseManagement.RegisterContainerProcessingParam {
        sectionFuncRank = "D",        --// Rank of base development function
        commonMetalTimeMinute = 25,   --// Time in minutes
        minorMetalTimeMinute = 25,    --// Time in minutes
        preciousMetalTimeMinute = 25, --// Time in minutes
        fuelResourceTimeMinute = 25,  --// Time in minutes
        bioticResourceTimeMinute = 25 --// Time in minutes
    }

GLOBAL proccessing rates (of raw materials)
    
    // This table effects ALL bases. These base amounts are redistributed by the engine.
    // If my total is 1000 for all counts. Fuel being 100 and minor being 100, there is a chance ~5% (somewhere in 5%) is added/removed from each value. So, fuel output may be 95 and minor comes out to 105.
    // This is done to vary the outputs so it's not always the same per material.
    TppMotherBaseManagement.RegisterContainerProcessingBasicParam {
        commonMetalProcessCount = 2500,    --// Base amount of unrefined materials (max 2500 offline, 5000 online)
        minorMetalProcessCount = 375,     --// Base amount of unrefined materials (max 2500 offline, 5000 online)
        preciousMetalProcessCount = 75,   --// Base amount of unrefined materials (max 2500 offline, 5000 online)
        fuelResourceProcessCount = 750,   --// Base amount of unrefined materials (max 2500 offline, 5000 online)
        bioticResourceProcessCount = 750, --// Base amount of unrefined materials (max 2500 offline, 5000 online)
        fobRate = .15                     --// Percent added by each FOB. Essentially, if it's .15, each FOB will add ~15% to each value, at it's given rank and output time. (idk why Konami decided to use a decimal, all the other rates don't have it.)
    }

RESOURCE TYPES

    FuelResource
    CommonMetal
    BioticResource
    MinorMetal
    PreciousMetal

Individual resources

    // This registers the resource to the base system. The GMP value is also set here.
    // Appears to be have max of 100 which is hardcoded into the engine.
    TppMotherBaseManagement.RegisterResourceParam {
        resource = "CommonMetal", --// Type
        baseSalePrice = 10,       --// Value per unit
        countInContainer = 100,   --// ???
        countPer1Minute = 4       --// ???
    }

Shipping containers

    TppMotherBaseManagement.RegisterContainerParam {
        commonMetalCounts = {
            white = 2e3, --// White (abundant)
            red = 6e3,   --// Red (Rare)
            yellow = 4e3 --// Yellow (common)
        },
        fuelResourceCounts = {
            white = 6e3,  --// White (abundant)
            red = 24e3,   --// Red (Rare)
            yellow = 12e3 --// Yellow (common)
        },
        bioticResourceCounts = {
            white = 1e3, --// White (abundant)
            red = 3e3,   --// Red (Rare)
            yellow = 2e3 --// Yellow (common)
        },
        minorMetalCounts = {
            white = 200, --// White (abundant)
            red = 600,   --// Red (Rare)
            yellow = 400 --// Yellow (common)
        },
        preciousMetalCounts = {
            white = 100, --// White (abundant)
            red = 300,   --// Red (Rare)
            yellow = 200 --// Yellow (common)
        },
        usableResourceContainerRate = 60, --// White (abundant) container spawn rate
        redContainerCountRate = 40,       --// Red (Rare) container spawn rate
        yellowContainerCountRate = 50     --// Yellow (common) container spawn rate
    }