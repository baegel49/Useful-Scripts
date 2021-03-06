Batch Gear Modification in Eden
===============================
**The following section describes a skeleton you need to use in combination
with anything further down below, the examples are not usable on their own.**
For easy video guide, see <https://www.youtube.com/watch?v=ajFO3cFKKr8>.

These code snippets allow you to modify already existing Arsenal-customized
loadouts on existing units, modifying their gear in batch, so that you don't
have to do it manually for each one.

The functionality is as if you edited the loadouts for all units manually
via Arsenal, just less tedious. :)

The basic idea is to go over all **selected** units using this loop:
```
{
    private _unit = _x;

    ...

} forEach get3DENSelected "object";
save3DENInventory get3DENSelected "object";
```
where the `...` should be replaced by whatever you want to do (see below).
You also need to save the new loadout to the mission file, hence the
`save3DENInventory`.

For example, if you wanted to add NVGs to all units (first line) and give them
suppressors (second line) and lasers (third line), you would do it by selecting
all playable units, pressing Ctrl-D to open up the debug console, pasting the
following into the big box and pressing the "LOCAL EXEC" button:
```
{
    private _unit = _x;
    _unit linkItem "rhsusf_ANPVS_15";
    _unit addPrimaryWeaponItem "rhsusf_acc_rotex5_grey";
    _unit addPrimaryWeaponItem "acc_pointer_IR";
} forEach get3DENSelected "object";
save3DENInventory get3DENSelected "object";
```

Zeus version
------------
You can use the same logic when in Zeus using the Debug console (Esc) if you
are logged in as Admin. Simply select units with mouse like you would do
in Eden, then use this skeleton instead of the above one:
```
{[_x, {
    private _unit = _this;

    ...

}] remoteExec ["call", _x]} forEach (curatorSelected select 0);
```

Execute Code (Eden) version
---------------------------
We have a custom editor module that can execute code directly in Eden when it's
placed (or the OK button is pressed when editing it). This can be used to
automate batch changes by saving the module, with code pre-configured, as a custom
composition, so that mere placing it will run the code.

Since we can't select any units when placing the composition, we have to simply
rely on changing all "playable" units, which isn't ideal, but should work for you.
Use this snippet:
```
{
    private _unit = _x;

    ...

} forEach playableUnits;
save3DENInventory playableUnits;
```

Things to replace `...` with
============================

NVGs
----
To add:
```
_unit linkItem "classname_here";
```

To remove existing (if unit has one):
```
_unit unlinkItem hmd _unit;
```

Weapon optic/suppressor/bipod/flashlight/laser
----------------------------------------------
In the following commands,

  * main weapon is `PrimaryWeapon`
  * launcher is `SecondaryWeapon`
  * handgun is `Handgun`

and for brevity, the examples below are only for `PrimaryWeapon`,
but work on all three, just adjust the names.

To add,
```
_unit addPrimaryWeaponItem "classname_here";
```

To remove existing by class name,
```
_unit removePrimaryWeaponItem "classname_here";
```
to remove item from a specific slot (0 = suppressor, 1 = flash/laser, 2 = optic, 3 = bipod):
```
_unit removePrimaryWeaponItem (primaryWeaponItems _unit select 0);
```
or to remove all
```
removeAllPrimaryWeaponItems _unit;
```

Binoculars
----------
In Arma, binoculars are considered a "weapon". This means you cannot add them
as an item using `linkItem`, but have to use `addWeapon`.
```
_unit addWeapon "classname_here";
```
If a unit already has binoculars, the command is ignored, though. To remove
any possible binoculars and force your own,
```
_unit removeWeapon binocular _unit;
_unit addWeapon "classname_here";
```
Because binoculars are weapons, some need a "magazine" (like a battery) to
work, like the laser designator. In this case, add the battery before the
"weapon" itself:
```
_unit addMagazine "Laserbatteries";
_unit addWeapon "Laserdesignator";
```
(add `removeWeapon` on top, as shown above, to remove any existing binocular)

Sometimes, you want to only swap binoculars, meaning to add only if the unit
already has one, like when switching Vector 21 Day for the Nite version:
```
if (binocular _unit == "ACE_VectorDay") then {
    _unit removeWeapon binocular _unit;
    _unit addWeapon "ACE_Vector";
};
```
or if you don't know what binocular classname the unit has, but you want to
remove it and force your own, without adding it to binocular-less units:
```
if (binocular _unit != "") then {
    _unit removeWeapon binocular _unit;
    _unit addWeapon "rhs_tr8_periscope";
};
```

UGL Flares instead of UGL Smokes
--------------------------------
Just copy-paste all of the following, the code is smart enough to replace the
vanilla smokes as well as RHS russian smokes and looks through the backpack
as well as any loaded primary/handgun shells (ie. for PLT lead).

Just imagine all of the code is a single line, like the ones above, and put
it where the `...` would be. You can of course add other things before/after
it.
```
{
    _x params ["_old", "_new"];
    private _cnt = { _x == _old } count backpackItems _unit;
    while {_old in backpackItems _unit} do { _unit removeItemFromBackpack _old };
    while {_cnt > 0 && {_unit canAddItemToBackpack [_new, 1]}} do {
        _unit addItemToBackpack _new;
        _cnt = _cnt - 1;
    };
    if (_old in primaryWeaponMagazine _unit) then {
        _unit removePrimaryWeaponItem _old;
        _unit addPrimaryWeaponItem _new;
    };
    if (_old in secondaryWeaponMagazine _unit) then {
        _unit removeSecondaryWeaponItem _old;
        _unit addSecondaryWeaponItem _new;
    };
} forEach [
    ["1Rnd_Smoke_Grenade_shell", "UGL_FlareWhite_F"],
    ["1Rnd_SmokeRed_Grenade_shell", "UGL_FlareRed_F"],
    ["1Rnd_SmokeGreen_Grenade_shell", "UGL_FlareGreen_F"],
    ["1Rnd_SmokeYellow_Grenade_shell", "UGL_FlareYellow_F"],
    ["1Rnd_SmokePurple_Grenade_shell", "UGL_FlareWhite_F"],
    ["1Rnd_SmokeBlue_Grenade_shell", "UGL_FlareGreen_F"],
    ["1Rnd_SmokeOrange_Grenade_shell", "UGL_FlareWhite_F"],
    ["rhs_GRD40_White", "rhs_VG40OP_white"],
    ["rhs_GRD40_Green", "rhs_VG40OP_green"],
    ["rhs_GRD40_Red", "rhs_VG40OP_red"]
];
```

Replacing primary weapon + mags
-------------------------------
The following is an example how to automate weapon swapping. Here, I want to
give players the AAC "Honey Badger" weapon, but because it doesn't have an UGL,
I want to provide a separate M320 for soldiers who had UGL-capable weapons.

As usual, all of the examples below go in place of the `...` in the loop
introduced at the beginning of this document.

Note how incredibly similar the code is to an Arsenal export.

```
{ _unit removeMagazines _x } forEach getArray (configFile >> "CfgWeapons" >> primaryWeapon _unit >> "magazines");
if (count primaryWeaponMagazine _unit > 1) then {
    { _unit removeMagazines _x } forEach getArray (configFile >> "CfgWeapons" >> handgunWeapon _unit >> "magazines");
    _unit removeWeapon handgunWeapon _unit;
    _unit addWeapon "rhs_weap_M320";
    _unit addHandgunItem "1Rnd_HE_Grenade_shell";
};
_unit removeWeapon primaryWeapon _unit;
_unit addWeapon "hlc_rifle_honeybadger";
_unit addPrimaryWeaponItem "muzzle_HBADGER";
_unit addPrimaryWeaponItem "29rnd_300BLK_STANAG";
for "_i" from 1 to 8 do { _unit addItem "29rnd_300BLK_STANAG" };
for "_i" from 1 to 2 do { _unit addItem "29rnd_300BLK_STANAG_T" };
```
In English,
  * remove any existing magazines usable for the existing primary weapon
  * if the weapon has more than 1 "magazine" inside it (assume it has an UGL)
    * remove handgun magazines usable for the existing handgun
    * remove the handgun
    * add standalone M320 to the handgun slot
    * load HE round into it
  * remove the primary weapon
  * add our new primary weapon
  * give it a suppressor
  * load a magazine into it
  * add 8 more magazines somewhere into inventory
  * add 2 more tracer magazines into inventory

My hope is that you can roughly understand how to modify and copy/paste bits of
the code around to suit your needs - ie. if your replacement weapon has an UGL
variant, you would add it "inside" the `if` block while adding the non-UGL
inside an `else` block,
```
<code which both UGL and non-UGL share>
if (count primaryWeaponMagazine _unit > 1) then {
    <code related only to soldiers with UGL>
} else {
    <code related only to soldiers without UGL>
};
<code which both UGL and non-UGL share>
```
and whatever is inside `else` can be further conditioned using a nested `if`.

For example, suppose we have a UGL-capable version of our weapon (so no need
for the handgun M320 hack from above) and we want to differentiate between AR
soldier and a regular soldier:

```
{ _unit removeMagazines _x } forEach getArray (configFile >> "CfgWeapons" >> primaryWeapon _unit >> "magazines");
if (_unit ammo primaryWeapon _unit >= 50) then {
    _unit removeWeapon primaryWeapon _unit;
    _unit addWeapon "hlc_lmg_MG42";
    _unit addPrimaryWeaponItem "hlc_50Rnd_792x57_B_MG42";
    while {_unit canAdd "hlc_50Rnd_792x57_B_MG42"} do { _unit addItem "hlc_50Rnd_792x57_B_MG42" };
} else {
    if (count primaryWeaponMagazine _unit > 1) then {
        _unit removeWeapon primaryWeapon _unit;
        _unit addWeapon selectRandom ["hlc_rifle_auga3_GL_B", "hlc_rifle_auga3_GL_BL", "hlc_rifle_auga3_GL"];
        _unit addPrimaryWeaponItem "1Rnd_HE_Grenade_shell";
    } else {
        _unit removeWeapon primaryWeapon _unit;
        _unit addWeapon selectRandom ["hlc_rifle_auga3_b", "hlc_rifle_auga3_bl", "hlc_rifle_auga3"];
    };
    _unit addPrimaryWeaponItem "hlc_30Rnd_556x45_SOST_AUG";
    _unit addPrimaryWeaponItem "ACE_muzzle_mzls_L";
    for "_i" from 1 to 8 do { _unit addItem "hlc_30Rnd_556x45_SOST_AUG" };
    for "_i" from 1 to 2 do { _unit addItem "hlc_30Rnd_556x45_T_AUG" };
};
```
In English,
  * remove any existing magazines usable for the existing primary weapon
  * if the soldier has 50 or more loaded rounds, assume he has an AR weapon
    * remove the primary AR weapon
    * add our own AR weapon
    * load a box into it
    * fill the inventory with as many ammo boxes as possible
  * else the weapon has 49 or less rounds, so we assume it's a non-AR weapon
    * if the weapon has more than 1 "magazine" inside it (assume it has an UGL)
      * remove the primary weapon
      * add a random UGL-capable primary weapon out of the 3 listed (black/blue/green) :)
      * load a HE shell into it
    * else the weapon is a non-AR non-UGL normal rifle
      * remove the primary weapon
      * add a random primary weapon out of the 3 listed (black/blue/green)
    * load the primary weapon with a magazine
    * give the primary weapon an ACE flash suppressor
    * add 8 more magazines into inventory
    * add 2 more tracer magazines into inventory

Of course I hope you won't ever need code this complex, but I wanted to
show-off some of the nice copy-paste-able examples you could use in simpler
scripts.

Theoretically, you could keep your weapon sets separate from "factions" or
compositions in this format (as you can then apply the code to any group,
of any side and any faction), but that's up to you.

And as always, anything above can be combined with any of the other code
snippers from this document (like adding NVGs).

Changing Uniform/Vest/Backpack/etc.
-----------------------------------
Normally, removing and adding a uniform/vest/backpack would clear the
contents, however we can "cheat" around that by saving the entire loadout
aside (to a variable), modifying just the classname itself, and restoring
the loadout back. This can be done individually for each piece, but makes
more sense if we can save once, modify more things at the same time and then
restore the loadout back with all changes applied.

The `if` checks are there in case a unit has no uniform/vest/backpack, so
that we only swap an existing piece for our one, not add anything.

For more info on the loadout array format,
[see the BI wiki](https://community.bistudio.com/wiki/Talk:getUnitLoadout).

To change just uniform:
```
private _gear = getUnitLoadout _unit;
if (count (_gear select 3) > 0) then {
    (_gear select 3) set [0, "classname"];
};
_unit setUnitLoadout _gear;
```
Just vest:
```
private _gear = getUnitLoadout _unit;
if (count (_gear select 4) > 0) then {
    (_gear select 4) set [0, "classname"];
};
_unit setUnitLoadout _gear;
```
Just backpack:
```
private _gear = getUnitLoadout _unit;
if (count (_gear select 5) > 0) then {
    (_gear select 5) set [0, "classname"];
};
_unit setUnitLoadout _gear;
```
Set all of uniform/vest/backpack:
```
private _gear = getUnitLoadout _unit;
if (count (_gear select 3) > 0) then {
    (_gear select 3) set [0, "uniform_classname"];
};
if (count (_gear select 4) > 0) then {
    (_gear select 4) set [0, "vest_classname"];
};
if (count (_gear select 5) > 0) then {
    (_gear select 5) set [0, "backpack_classname"];
};
_unit setUnitLoadout _gear;
```

As a specific example, let's change the uniform/vest/backpack/helmet to cnto
custom winter flecktarn camo:
```
private _gear = getUnitLoadout _unit;
if (count (_gear select 3) > 0) then {
    (_gear select 3) set [0, "cnto_flecktarn_u_snow"];
};
if (count (_gear select 4) > 0) then {
    (_gear select 4) set [0, selectRandom ["cnto_flecktarn_v_l_snow", "cnto_flecktarn_v_snow", "cnto_flecktarn_v_h_snow", "cnto_flecktarn_v_s_snow"]];
};
if (count (_gear select 5) > 0) then {
    private _bpclass = (_gear select 5) select 0;
    private _bpmaxload = getNumber (configFile >> "CfgVehicles" >> _bpclass >> "maximumLoad");
    switch (_bpmaxload) do {
        case 160: { (_gear select 5) set [0, "cnto_flecktarn_b_ap_snow"] };
        case 280: { (_gear select 5) set [0, "cnto_flecktarn_b_kb_snow"] };
    };
};
_gear set [6, selectRandom ["cnto_flecktarn_h_6b27m_snow", "cnto_flecktarn_h_6b27me_snow"]];
_unit setUnitLoadout _gear;
```
In English,
  * save the current loadout to a variable, so we can work on it
  * if the unit has a uniform (is not an empty array)
    * set its classname to ours
  * if it has a vest (not an empty array)
    * choose randomly one from the 4 specified and use it
  * if the unit has a backpack (not an empty array)
    * get the current backpack classname
    * read its `maximumLoad` value, how much it can carry
    * switch (pick) based on this value a suitable replacement - we don't care
      what the soldier currently has, only that we pick a backpack large enough
      to fit the contents
      * 160 is maximum load of an assault pack, so use our retextured AP
      * 280 is load of a kitbag, so use our retextured KB
  * set the headgear (helmet) to a random selection out of 2 classes specified
  * restore the modified loadout into the soldier

Randomizing variants of a weapon (color)
----------------------------------------
If a weapon has ie. 4 variations on stock, magazine looks, left hand grip,
color, camo, etc. and you don't want everyone to have the same weapon, use
the following to shake things up a bit,
```
private _gear = getUnitLoadout _unit;
if (count (_gear select 0) > 0) then {
    (_gear select 0) set [0, selectRandom ["class1","class2","class3"]];
};
_unit setUnitLoadout _gear;
```
specifying all weapon classes you want to pick from (incl. the original
weapon). All attachments, magazines / shells, etc. are preserved.

To conditionally replace weapons (so that you don't swap non-UGL for UGL
or vice versa), you can limit the choices based on how many "magazine"
types the weapon has (assuming >1 is UGL), see replacing primary weapon
with magazines in the section above.
```
private _gear = getUnitLoadout _unit;
if (count (_gear select 0) > 0) then {
    if (count primaryWeaponMagazine _unit > 1) then {
        (_gear select 0) set [0, selectRandom ["rhs_weap_m4a1_carryhandle_m203","rhs_weap_m4a1_carryhandle_m203S"]];
    } else {
        (_gear select 0) set [0, selectRandom ["rhs_weap_m4a1_carryhandle_pmag","rhs_weap_m4a1_carryhandle_mstock"]];
    };
};
_unit setUnitLoadout _gear;
```

Switching forest/snow/grassland/mediterranean/etc. variants
-----------------------------------------------------------
Sometimes, you'll have gear pieces that come in several flavours, such as
the cnto flecktarn uniforms (or ie. vanilla backpacks). These have identical
classnames except for the variant, ie.
```
cnto_flecktarn_u_forest
```
versus
```
cnto_flecktarn_u_snow
```
Let's also assume you have multiple loadout pieces like this (uniforms, vests,
helmets, hats, caps, facewear, backpacks, weapons, etc.) and within each type,
you can have further variation (vest type - light/heavy/special/etc., headgear -
hat, cap, etc., facewear - bandana, balaclava, etc.). Replacing all of this
manually would be a nightmare.

Let's also assume you don't want to use the random approach from above, you
want soldiers who had light vest to also have a light vest in the new variant.

So what if we took all of the loadout, all classnames, and in each of them,
regardless of item type, replaced one string piece ("forest") for another
("snow"), keeping everything else?

The following code does just that:
```
private _replace_strings = {
    params ["_from", "_to", "_gear"];
    {
        if (_x isEqualType []) then {
            [_from, _to, _x] call _replace_strings;
        } else {
            if (_x isEqualType "" && {_x find "cnto_flecktarn_" == 0}) then {
                private _off = _x find _from;
                if (_off != -1) then {
                    _gear set [_forEachIndex, (_x select [0, _off]) +
                               _to + (_x select [_off + count _from])];
                };
            };
        };
    } forEach _gear;
};

private _new_camo = "snow";

private _gear = getUnitLoadout _unit;
["mediterranean", _new_camo, _gear] call _replace_strings;
_unit setUnitLoadout _gear;
```
This replaces any gear piece which contains "mediterranean" in its class name
with a variant which contains "snow" in the same place of the class name.

The following is the same code, with weapon swapping (based on original class)
added:
```
private _replace_strings = {
    params ["_from", "_to", "_gear"];
    {
        if (_x isEqualType []) then {
            [_from, _to, _x] call _replace_strings;
        } else {
            if (_x isEqualType "" && {_x find "cnto_flecktarn_" == 0}) then {
                private _off = _x find _from;
                if (_off != -1) then {
                    _gear set [_forEachIndex, (_x select [0, _off]) +
                               _to + (_x select [_off + count _from])];
                };
            };
        };
    } forEach _gear;
};
private _swap_weapons = {
    params ["_camo", "_gear"];
    private _weaparr = _gear select 0;
    if (count _weaparr < 1) exitWith {};
    private _new = switch (_camo) do {
        case "desert": {
            switch (_weaparr select 0) do {
                case "rhs_weap_hk416d10":       { "cnto_hk416_d10" };
                case "rhs_weap_hk416d10_m320":  { "cnto_hk416_d10_m320" };
                case "rhs_weap_hk416d145":      { "cnto_hk416_d145" };
                case "rhs_weap_hk416d145_m320": { "cnto_hk416_d145_m320" };
                default {};
            };
        };
        case "snow": {
            switch (_weaparr select 0) do {
                case "rhs_weap_hk416d10":       { "cnto_hk416_d10s" };
                case "rhs_weap_hk416d10_m320":  { "cnto_hk416_d10_m320s" };
                case "rhs_weap_hk416d145":      { "cnto_hk416_d145s" };
                case "rhs_weap_hk416d145_m320": { "cnto_hk416_d145_m320s" };
                default {};
            };
        };
        default {};
    };
    if (!isNil "_new") then {
        _weaparr set [0, _new];
    };
};

private _new_camo = "snow";

private _gear = getUnitLoadout _unit;
["mediterranean", _new_camo, _gear] call _replace_strings;
[_new_camo, _gear] call _swap_weapons;
_unit setUnitLoadout _gear;
```

Replacing unique instances of ACRE2 radios with generic ones
------------------------------------------------------------
The following replaces classnames like `ACRE_PRC343_ID_5` with the more
generic `ACRE_PRC343`. Useful in case you saved an Arsenal loadout in
a running game and want to use it in Eden. Radio order in the inventory
is preserved.
```
private _replace_radios = {
    {
        _x params ["_class", "_cnt"];
        if (_class find "ACRE_" == 0) then {
            private _idpos = _class find "_ID_";
            if (_idpos > 0) then {
                private _newclass = _class select [0,_idpos];
                _x set [0, _newclass];
            };
        };
    } forEach _this;
};

private _gear = getUnitLoadout _unit;
_gear params ["_priweap", "_launcher", "_handgun",
              "_uniform", "_vest", "_backpack"];
if (count _uniform > 0) then {
    (_uniform select 1) call _replace_radios;
};
if (count _vest > 0) then {
    (_vest select 1) call _replace_radios;
};
if (count _backpack > 0) then {
    (_backpack select 1) call _replace_radios;
};
_unit setUnitLoadout _gear;
```
