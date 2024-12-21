# NSFW Toggle INI Library
This library INI was created initially as a way to have an in-game toggle for NSFW mods without needing to rename the folders themselves. It is used in conjunction with the idea of having a Master INI file for each character (rather each character skin to be more precise) but does not have to be. What follows is an explanation of how to utilize this library both for the Master INI system and for individual mods.
> [!IMPORTANT]
> Depends on files from XXMI Core for printing notifications at the moment. Editing to support users of model importers independent of XXMI may come at a later time.
### Master INI System
With a Character Master INI, you gain the ability to swap between active mods without having to disable folders to ensure that only one mod for a character is active at a time. This also eliminates the need for using simple external mod-managers; however I should note there are a number of these tools that provide valuable additional functionality which you may consider still worth using in other capacities.

The Master INI would need to implement something like the example implementation below with the following assumptions:
* there are 6 mods for the character and we use -1 for "no mod"
* the sfw mods are everything except the 2nd and 5th (swapvar values 1 and 4)
* the 5th mod is a toggle mod with sfw variant(s)
* you decide to identify this character as one that belongs to one of the available categories coded as 'femalensfw'

<details>
<summary>Example implementation</summary>

```ini
[Constants]
global persist $swapvar = 0
global persist $lastsfw = 0
global persist $lastnsfw = 0
global persist $isnsfw = 0
global $nsfwenabledcopy = 1
global $nsfwtoggleval
global $active

[KeySwapSFW]
condition = $active == 1 && $\global\nsfwtoggle\femalensfw == 0
key = ctrl VK_RIGHT no_shift no_alt
back = ctrl VK_LEFT no_shift no_alt
type = cycle
$swapvar = -1,0,2,3,4,5
$isnsfw  =  0

[KeySwapSFW.2]
condition = $active == 1 && $\global\nsfwtoggle\femalensfw == 1 && $isnsfw == 0
key  = ctrl VK_UP no_shift no_alt
back = ctrl VK_DOWN no_shift no_alt
type = cycle
$swapvar = -1,0,2,3,5
$isnsfw  =  0

[KeySwapNSFW]
condition = $active == 1 && $\global\nsfwtoggle\femalensfw == 1 && $isnsfw == 1
key = ctrl VK_UP no_shift no_alt
back = ctrl VK_DOWN no_shift no_alt
type = cycle
$swapvar = 1,4

[KeySwapFull]
condition = $active == 1 && $\global\nsfwtoggle\femalensfw == 1
key = ctrl VK_RIGHT no_shift no_alt
back = ctrl VK_LEFT no_shift no_alt
type = cycle
$swapvar = -1,0,1,2,3,4,5
$isnsfw  =  0,0,1,0,0,1,0

[KeySelectSFW]
condition = $active == 1 && $\global\nsfwtoggle\femalensfw == 1 && $isnsfw == 1
key = shift VK_RIGHT no_ctrl no_alt
back = shift VK_LEFT no_ctrl no_alt
type = cycle
$lastsfw = -1,0,2,3,4,5

[KeySelectNSFW]
condition = $active == 1 && $\global\nsfwtoggle\femalensfw == 0 && $isnsfw == 0
key = shift VK_RIGHT no_ctrl no_alt
back = shift VK_LEFT no_ctrl no_alt
type = cycle
$lastnsfw = 1,4

[KeyToggleLastSaved]
condition = $active == 1 && $\global\nsfwtoggle\femalensfw == 1
key = shift . no_alt no_ctrl
run = CommandListToggleSaved

[Present]
post $active = 0
$nsfwtoggleval = $\global\nsfwtoggle\femalensfw
$shouldswap = ($nsfwenabledcopy != $nsfwtoggleval) && ($nsfwtoggleval == 0 || $\global\nsfwtoggle\notificationtype != 1)

if $nsfwtoggleval == 0
    if $isnsfw
        $lastnsfw = $swapvar
        if $shouldswap
            $swapvar = $lastsfw
            $isnsfw = 0
        endif
    endif
else if $nsfwtoggleval == 1
    if !$isnsfw
        $lastsfw = $swapvar
        if $shouldswap
            $swapvar = $lastnsfw
            $isnsfw = 1
        endif
    endif
endif
$nsfwenabledcopy = $nsfwtoggleval

[TextureOverrideCharacterPosition]
hash = <hash>
$active = 1

[CommandListToggleSaved]
if $isnsfw
    $lastnsfw = $swapvar
    $swapvar = $lastsfw
    $isnsfw = 0
else
    $lastsfw = $swapvar
    $swapvar = $lastnsfw
    $isnsfw = 1
endif
```
</details>

Obviously the actual toggle keys you want to use can be changed, but I've found these to be good and not used much by existing mods.

By setting things up like this, you are now able to maintain your last used SFW and NSFW mod even if that happens to be the same mod, and freely toggle between the two or to be able to toggle all NSFW off or just a single category of NSFW off and have this character instantly swap to its last SFW mod (or load in with it if you switch to that character later).
### For individual mods
Authors of mods and users willing to edit INI files could make use of this library by offering a SFW reset and the ability to save the values of individual part variables if the toggle mod is Outfit Compiled properly for quick restoration of the user's loadout before the reset. Old-style merge mods which use a single `swapvar` variable can also take the above `lastsfw`/`lastnsfw` approach but without the hotkeys as those are more for the above Character management system using a Character Master INI.

The latter case of old-style merge mods is much easier to implement as you copy the same code from the `[Present]` block seen above and ensure the variables are declared in the `[Constants]` section.

The former case, however, cannot be generalized. For those types of mods it is best to analyse which toggles result in NSFW outcomes and restrict either the hotkey condition, the Override with the relvant `drawindexed` calls, or the CommandList with relevant code to check for the value of the NSFW variable for that category of character. Complex toggle mods that have UI menus will be even more tedious to edit.
### Available Categories
At the moment, the following categories are implemented in the INI and available for use in mods:

* `$femalensfw`
* `$malensfw`
* `$lolinsfw`
* `$shotansfw`

There is also the catch-all `$nsfw` which should not generally be used as there is internal logic keeping this value in sync with the categories but it can be if you *don't* use any of the categories, and the meta-group `$lolishonsfw` which similarly should not be used.

> [!NOTE]
> Yes, yes, I know what you're likely thinking. My stance is that as a tool it holds no opinions and should not be made opinionated in what types of mods it supports or doesn't support. The fact is simply there are users of these kinds of mods and they could easily be users of this tool too.

There are currently no plans to add more categories although I have thought about trying to include one for so-called "Hard NSFW"; the very simple reason I haven't yet being that I am *out* of hotkeys. Seriously, look in the file... I have exhausted the reasonable options and even those alone took me, and I am not exaggerating, a collective 3 hours of thought to finalize. I'd rather not waste more time on that.
