# SLIGE
SLIGE Doom WAD Generator source code by Dave M. Chess (https://www.doomworld.com/slige/)

  -readme-

   SLIGE - a random-level generator for DOOM
           by David M. Chess
           dmchess@aol.com         http://www.davidchess.com/
                                   http://users.aol.com/dmchess/
           chess@watson.ibm.com    http://www.research.ibm.com/people/c/chess/

           http://www.doomworld.com/slige/

   January, 2000

   This source code is made available to the world in general,
   with the following requests:

   - If you're doing a port and you find that something or other needs
     twiddling to get it to compile, let me know and I'll stick in some
     ifdefs.  The current code assumes that Microsoft C has a horrible
     rand() function and uses str(n)icmp() instead of str(n)casecmp(),
     whereas everyone else has an ok rand() and str(n)casecmp().  And
     make sure that you compile with tight structure-packing (/Zp1 or
     -fpack_struct, I think) or DOOM won't understand the files.  Also,
     if you want to port to a non-Intel-byte-order platform, you've got
     a bit of work, but I think it's mostly isolated to DumpLevel,
     CloseDump, dump_texture_lmp, and a few others.  Good luck!  *8)

   - The current version should compile without any special switches
     except structure-packing, under both MSVC++ and DJGPP, without
     any warnings (unless you use -O3 in DJGPP, in which case you'll
     get some warnings that don't matter).  It should also compile
     and work under Linux gcc, but I've never tried it myself; if you
     do, write and tell me about it!  Anyone wanna do a Mac port?
     I'd be glad to give advice...

   - If you make various changes and improvements to it and release
     a modified version yourself:

     - Write me and tell me about it so I can be pleased,
     - Mention me and SLIGE in the docs somewhere, and
     - Please *don't* call your new program "SLIGE".  SLIGE is
       my program.  Call yours "BLIGE" or "EGGISLES" or "MUMFO"
       or something like that.

   - If you want to do lots of wonderful things to the program to
     make it better, and then send me the result to incorporate into
     my next version, I strongly suggest telling me your plans first.
     Otherwise you may hear nothing from me for weeks after sending me
     your source mods (I don't check my mail that often sometimes), and
     then get a disappointing "thanks, but I don't want to take the
     program that direction, I have other plans" sort of reply.  Small
     bug fixes, of course, are always welcome!  All source mods sent
     to me become the property of the world at large.

   Otherwise, enjoy the program, and let me know what you think of the code!

   DC

*/
#define SOURCE_SERIAL (485)
/*

   New stuff (since 474): first try at random DM starts/weapons, fix rare
     trap in falling core generation, add -nosemo switch, fix(?) bug from
     rellwood that could get you trapped on a step-dias, -bimo/!/we switches,
     no jambs without doors, remove locked gates in DM (per Jim Kneuper),
     make extra-huge more likely in DM.

   Stuff to do:
     BUG: if there's a gate leading to the room containing the gate leading
       to the arena, that first-mentioned gate ?sometimes doesn't work;
       see slige (475) -e2m1 -config blue.cfg -arena -seed 1098402211.
       (Looks like the in-between gate has no sector tag and no TP exit in it.)
     BUG: very rarely, a slige -dm level will still not have four DM starts.
       Try more rooms than just first and last, and/or use a smaller "width"
       to allow putting a DM start on a pickable object?
     fold in the RISC OS mods from Justin Fletcher,
       SPARC mods from Oliver Kraus
     fold in Rob Ellwood's ceiling-effect and split_linedef mods,
       when he has a version he likes enough
     armor-damage model is wrong: green armor only absorbs 1/3 of
       damage taken; only blue armor absorbs 1/2.  Fix!
     BUG: when trigger_box() calls point_sector(), it should check
       the "danger" return (rather than passing in NULL)
     more lamps/lights when night and/or dim?
     very-dark secret-level flavor?
     level flavor like "always big floordeltas"
     implement the "favorite monster" style of secret level
     simple multi-level "ramps" in patios, for variety?
     sometimes monsters can be so tightly packed into a small room
       that none can move until you kill one.  Detect/avoid somehow.
     eliminate "ladders", by having random_basic_link() just make sure that
       if there are stairs abs(floordelta)<=linkdepth?
     switches mounted on columns/placards, not just in walls
     nukage in downsecs of falling-core traps, sometimes.  Barrels, too!
     regularize lighting (less ad-hoc, more constants), have dark
       secret levels, perhaps darker style, perhaps dark rooms with
       bright otherstuff, etc.
     secret doors/closets at the ends of ambush closets
     configify lamps and barrels
     configify other objects and monsters and anything else that's left
     unhardcode health's amounts, weapons' damage, etc
     shouldn't "error" and "gateexitsign" just be property bits?
     fixed the bug that sometimes accidentally made the monster hiding
       on top of a pillar be down in the room; but it'd be cool to
       sometimes do this on purpose!
     trees can block patio doors; fix
     BUG: A raised teleporter right next to a step-up doorway can be
       inaccessible.  Fix!
     once in awhile a barrel in a secret, to discourage the "open and
       shoot without looking" trick.  heh heh.
     have use of lights (gridlights, lightstrips, kickplate lights, etc)
       be correlated per level or config or something, not random
     use CEILING+LIGHT in even more lighting effects (link to
       light_recesses, for instance, and maybe use in windows)
     do mancubus-special in MAP07 even if not last_mission
     do arach-special in MAP07 also (somehow)
     place_timely_something() and friends need an "is it a secret?"
       switch, then could be used in various secret places.
     Secret grid-pillars need something more significant on them!
     Biggest-monster mode in DOOM 1 gives lots and lots and lots and
       lots of spectres...  (if BIG is off, I suspect)  Problem?
     Still the occasional level with many too many rooms; heh!
     BUG: the embellishment of the outersec of a rising room
       can decide to triggerbox the key, even though the rising room
       has already boxed it.  Fix.  (Actually both boxes seem to work,
       even though they're exactly coextensive, but still...)
     BUG: have seen at least one extwindow that extended orthogonal to
       the direction it should have extended.  Very odd!
     BUG: still getting a monster stuck in a floor-lamp now and then?  how?
       also perhaps a monster stuck in a grid-room pillar (or was that two
       grid-room monsters stuck to each other?)  (Those may be fixed now.)
       Also once saw a Hell Knight stuck between a new-style "pillar" and
       a wall.  Looked like.
     BUG: Sometimes a room has two different gates, both in the midtile.
       Well, you know what I mean...  This causes trouble.  Fixed?
       Nope; still can happen with a gate to an arena.  Is that OK?
     More exit-to-secret-level secret kinds.
     More special things about secret levels.  A black-and-starry
       theme using custom patches?
     For co-op, put all the weapons that we assume the player has
       into the start room (with multiplayer-only bit set).
     For DM, put a DM start (with a multiplayer-only weapon nearly)
       in every Nth room?  This conflicts with the above; maybe -dms?
     It would be nice to not give up on a bath whenever a monster might
       get stuck in the edge.  Try *moving* the monsters, or the edges
       of the bath, instead?
     Make arenas even more interesting.  powerups/armor?  More scenarii!
     dead-gardens flavor
     More interesting sorts of GATE_GOAL-locked links (like just
       a grated door).  Also have the gate be one-way if the locked
       link is passible in the far-to-near direction.
     More Instaforks!  (Really just pushes, not forks at all)
     twinned links need to have their extra-painted-door bits and
       nukage-link status and stuff correlated, eh?
     read switches from config file
     constructs (computers, at least) hanging from the ceiling?
     more rational facings in grid-rooms (sort of toward the door)
     have the new-pillar probability in the style, and have it sometimes
       be very very high?
     more open-link stuff: gratings, locks, secrets, monsters, etc, etc
     sometimes have open(ish) links block sound, to avoid excess monster
       accumulation?  (Doesn't seem to be a problem?)
     open lifts sometimes activate at the top also
     sometimes blaze autodoors / lifts
     monsters/pickables in sidesectors of an open link
     maybe have a minimum-link-width thing (used only on non-door links,
       maybe?), usually zero, and sometimes (often when bigification is
       high) higher (like 128).  For a non-narrow level, but not hugeness
     put nukage-edge textures (bottom-aligned) around nukage cores (and
       normal nukage pools, eh?)  Need new attribute bit/thing
     put pillars with nukage-pourers (green gunk vents, red demon
       faces, various leaking pipes) in nukage pools
     use step-textures and/or wall textures for/instead of support
       textures in various other kickplate places (below doors?)
     put decorations up on the centers of some pillars (/constructs)
     I suspect it's possible for a sequence of new-style pillars to
       block off the center of the room (thus perhaps making it impossible
       to get a key).  Implement fix if so.
     probably don't want to put triggerboxes around candles, eh?  They
       aren't *really* pickable, and the player is unlikely to go around
       stepping on all of them for any reason.  Not a big deal.
     make the sunrooms and nukage-city effects less correlated (if nukage
       was forced, have sky lower-prob, and vice-versa)
     Other patio things:
       There are often no plants.  That "128" is possibly too big?
       Store floor and wall textures in style? or even level?
       Patios should count toward the level's (the quest's?) room count, eh?
       Patios would really make more sense as a kind of microquest, rather
         than a mere embellishment
       Make sure "roof" height is also big enough to accomodate the
         link->height1, and for that matter the door's height (twice),
         if there's a door
     in homogenize mode, that hack that will always add one more of the
       room's monsters if there's room for *any* monster can produce
       very painful results (the Revenant Brothers, in particular).
       Fix it?  Just separate out the "is there room for monster M in
       the model" check into a routine to call in homgen mode.
     homogenize_monsters, should be per-level, not per-WAD, eh?  or should it?
     once had a really really steep stairway with a monster on it.  The
       stairway was so narrow I couldn't get around the monster, but so
       steep that I couldn't see it to shoot it!  avoid?  (I actually
       managed to sneak around it eventually, but it got in lots of bites.)
       probably just put no link-guards on steep stairs, eh?
     y-offsets pretty good now; put some support texture in when two
       coalignable textures come together at a base-ceiling boundary?
       that's about the only place that y-offset problems remain
     some cool light-effect around pillars (new and old styles)
     find all monster-width assumptions (tough!), generalize or whtever
     merge secret-closet and plaque-closet code sections (diffs: plaques
       are recessed, and are sometimes not openable at all)
     way-cool recessed windows style
     traditional locked nop-DOOR3 sometimes in entry room?
     sometimes put a rising room at the end of tag-switch quests, too?
     allow embellishing far sides of links (w/lightboxes etc?)
     sometimes vertical lightstrips on walls / in corners (won't happen
       in Wod theme, because no "LIGHT" walls anymore; OK?)
     more extensive outdoor areas; need a theory
     non-rectangular rooms (lots of work there!)
     same texture on both sides of a door?
     before giving up finding a linedef to put a link on, split
       all too-big linedefs around the current room
     if a lift can be walked off of, leave out the WR-lower linedef
       at the top end.  sometimes
     raised sniper-closets
     make level-exits look even more obvious in DOOM I somehow?
     sometimes put keys in ambush closets, other special places
     sometimes have the back wall of a closet be a secret door (hint?)
     have big monsters / weapons (only) in later episodes / maps?
     use of has_chainsaw/berserk on !(FLIES|SHOOTS) working OK?
     use has_chaingun information for something.  remove wimp-bonus?
       chainguns tend to be ammo-inefficient; recognize?
     silly to put in the linedef box around an object before checking if
       there's enough room for the secret closet, eh?  fixed?
     have plaque-doors sometimes be triggered also
     more sophisticated backpack counting (depending on the weapon(s)
       the player has)
     model the invis and radsuit better?
     a big boss monster in a big secret room that lowers down when you
       take pickable.  also in the boss' room is the key/switch you need.
       have to make sure player has enough ammo/health first.  details?
     put a single window-grating in the center, not two half-invisible
       ones on the sides
     don't watermark the first room of non-first levels in a PWAD?
     keep them TLITE ceilings on door-recess sectors (and stuff);
       looks way cool!  (CEILING+LIGHT, now)
     autodoors and lifts are sort of boring currently (only 24-deep rec's).
       make them more interesting?
     plain-sky ceilings don't work, because if a nearby room has a
       slightly higher non-sky ceiling, or worse is a steep stairway
       goes up out of the room, it looks really dumb.  think about fixes.
       perhaps always raise the ceiling to be at least as high as all
       ceilings of attached rooms?  that simple?
     the support_misaligns stuff looks awful.  think about this some more.
     skyclosets are still silly-looking if you can stand in the end and
       look back at the "building".  Either fix somehow (how?), or use
       point_froms to put the skylight only at the far end of the closet,
       even in deep closets, so you can't see out from the end.  Also,
       the lip of the outer sides of the skycloset walls isn't necessary;
       would look best, I think, with a lip on the near side, and the
       other sides just ending blap like they used to.
     deathmatch starts, somehow?  Maybe deathmatches
       in each goal-end room, or something?  As well as the start.
       Other DM-friendly stuff, -deathmatch switch.
     sometimes use a light-wall texture on walls (especially flanking
       doors/links?)  But there aren't that many of them.
     have a "minwall" in config, to usually forbid those skinny little
       4-pel-wide walls and like that.  (done a different way, but we
       might want to think about something >8 as a config option)
     multiple snipers/ambushers in a single (wider) closet?
     OK to have population before embellishment?  Should prevent
       monsters getting stuck in the wall because of swelling, yes?
       also makes haa-model accesses during embellishment more
       logical (since probably one cleans out the wandering monsters
       first, then the embellished?)  probably need both pre-pop and
       post-pop embellishments eventually for something
     random-link still returns too wide sometimes; that whole routine
       needs to be re-thought and rationalized (fixed?)
     use real door-texture widths, not the current hack, to center faces
     niches: often symettrically around doors, with lights/monsters/items
       sometimes with (secret) doors, sometimes non-walkable (lights,
       airholes to the outside, portraits, etc)
     generalize lightstrips to include counters, plaques, computer
       displays, even those little closets with monsters and stuff.
     do basic stairs by first making the core and then splitting it up; this
       will help auto-alignment, and sort of rationalize things also
       (can use stairify(), with a little work).
     swelling a room moves out the apparent corners, and that interferes
       with later find_rec() calls used during population.  Uh-oh!!
       need true distance-from-point-to-line code.  Now that we're
       embellishing after populating, this isn't as bad.
     figure out how alignment works on left sidedefs, for split_linedef()
       (current code OK?)
     style setting for closet height?  variation here is nice, though
     more interesting window-borders (not just random) and widths
       very narrow "windows" can of course have lower floors and
       higher heights, since you can't walk through 'em anyway
       (just like gratings).  Think about how to do in style / link.
       (Slits sort of provide that now.)
     window stuff in Link, not in Style, eh?
     have generate_room_outline sometimes extend the starting
       linedef with another colinear linedef at one or both
       ends, so you can come into a room on a *long* side
     in intersect(), properly check for overlap of colinear segments.
     use texture-size data more in alignment, door-choice, etc, etc
     need a copy_link() that does small perturbations to an
       existing link (like copy_style())
     restore the ability for twinned links to have different
       door types?  Say!  How about allowing the two links
       of a twinned link to be *completely different?  Well,
       they have to have the same total depth and floordelta,
       anyway.  Hmm...
     on the other hand, it'd be logical if the door-types at
       either end of a link were the same.  That means having
       it in the link, not the room's style.  Sensible?  On
       the third hand, they needn't always be the same, and
       for that matter couldn't you have an alcove with a door
       at one end, and none at the other?
     allow different-size openings at either end of a link?
       (tapered stair-walls, e.g.)
     have special deliverers for weapons if c->weapons_are_special
       (and then sometimes set that TRUE)
     more global (per-level or per-config) restrictions on style;
       never moving jambs, always soundproof doors, doorceilings
       always/never copy room ceiling, etc, etc?  Also pillars
       and windows and gunkage.  (Extend "Nukage City" concept)
     if link.height1==0, use ceiling height of main room, eh?
     pick one (at least) 64-aligned 64x64 square, make it a sector,
       change the ceiling texture to some light, maybe raise or
       lower the ceiling a bit, and change the light level (up,
       usually), and perhaps make it blink.
     need more general object-copiers (for split_linedef etc)
     generalize the idea that a monster can provide a weapon?  (eventually,
       including in config-file etc)
     in basic philosophy, we should think more about making the individual
       room fancy.  as it is, the overall flavor is likely to always
       be a chain of simple rooms.  Which is OK, but limited.  Put
       another way, the monsters standing around in a room are dull;
       the interesting stuff are the monsters in traps, on ledges,
       behind gratings, etc.  Do lots of monster-placement during
       embellishment?
     or another way, it's too easy now, just running through rooms
       killing things.  make it harder!
       
-Dave M. Chess
February, 2000
