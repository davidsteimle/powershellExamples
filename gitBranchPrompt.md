# Git Branch in Your Prompt

First, determine if the current location is a repositpory by looking for ``.git``

* ``Get-Item``
    * ``-Force`` to see hidden
    * ``-ErrorAction Ignore`` to supress errors when location is not a repository
* If it is a repository, use ``git branch --show`` to get the name
* Return the name in the desired format

```powershell
function Get-GitBranch{
    if(Get-Item .git -Force -ErrorAction Ignore){
        # this is a repo
        $Query = git branch --show
    } else {
        # this is not a repo
    }
    if($Query){
        " ($Query)"
    }
}
```

Next, build your prompt
* ``(Get-Location).Path`` for current location
* Call ``Get-GitBranch`` and use its response
* Return prompt text

```Powershell
function Get-MyPrompt{
    $Prompt = "PS $((Get-Location).Path)$(Get-GitBranch)>"
    $Prompt
}
```

Use standard ``prompt`` to set the new prompt for the location.

```powershell
function prompt {
    Get-MyPrompt
}
```

If this is in your ``$PROFILE`` it will execute on shell launch, and change as you navigate around your system.
