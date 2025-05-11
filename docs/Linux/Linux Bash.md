## for
语法
```
for variable [in words]; do
	commands
done
```

示例
```
# for i in A B C D; do echo $i; done
A
B
C
D

# for i in {A..D}; do echo $i; done
A
B
C
D

# for i in B*.zip; do echo $i; done
Best Selection Songs.zip
Best Selection Songs001.zip
```