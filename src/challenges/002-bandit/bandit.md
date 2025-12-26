

bandit1
ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If

---

# level 2

dash as file name
solution:

1. `cat ./-`
2. `cat < -`
3. `rev -  | rev`
4. `more -`

answer:
> 263JGJPfgU6LtdEvgfWU1XP5yac29mFx

more information:
1. `touch -- -file`
2. `cat -- -file`
3. `rm -vf -- -file`

---

# level 3

spaces in filename
solution:
1. `cat ./"--spaces in this filename--"`
2. `cat -- --spaces\ in\ this\ filename--`

answer:
> MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx

---

# level 4

solution:
1. `cat inhere/...Hiding-From-You`

answer:
> 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ

---

# level 5

solution:
1.  
```bash
for f in -*; do
  echo "===== $f ====="
  cat -- "$f"
  echo
done
```

2. 

answer:
> 4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw

---

# level 6

solution:
```
find . -type f | wc -l			# 180
find . -size 1033c 				#./maybehere07/.file2
find . ! -executable  | wc -l	# 60

# mixed solution
find . -type f -size 1033c ! -executable


# other things
find . -type f -size 1033c ! -executable -exec file {} \;
```

answer:
> HWasnPhtq9AVKe0dmk45nxy20cvUa6EG

---

# level 7

solution:
```bash
find / -type f -size 33c -user bandit7 -group bandit6 2> /dev/null | xargs cat
```

answer:
> morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj

---

# level 8

solution:
```bash
cat data.txt | wc -l  # 98567

grep "millionth" data.txt
# OR
awk '{ for (i=1;i<=NF;i++) if ($i=="millionth") print $(i+1) }' data.txt
# OR
grep millionth data.txt | cut  -f2
#

```

answer:
> dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc

---

# level 9

solution:


answer:
> 

---
