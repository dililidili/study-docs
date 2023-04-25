#### 1.1 子集调用父级

```
<!--父级-->

<div>
	<test-child @testMethods="createTest($event)"></test-child>
</div>

<!--父级-方法-->
createTest(message){
	console.log(message)
}

<!--子级-->

<div>
	<el-button @click.native="submit">提交</el-button>
</div>

<!--子级-方法-->
submit(){
	this.$emit("testMethods","参数")
}
```

