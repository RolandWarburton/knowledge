# Grep and regex

### Using quotes in regex expressions
**Problem:** Regex expression not acting correctly compared to other online regex tools.

**Solution:** put your regex in quotes.
```
# WRONG
cat a.txt | grep -E word.*
# CORRECT
cat a.txt | grep -E 'word.*'
```

### Get colors in grep
**Problem:** no colors on matched regex.

**Solution:** use the ```--color``` tag. Perhaps make an alias for it in zsh to use it automatically.
```
#~/.zshrc
alias grep="grep --color"
```
