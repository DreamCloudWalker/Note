



## 只使用不可重入（非递归）的锁

```C++
std::mutex mutex_;
std::vector<Test> test_;

void post(const Test& t) {
	std::lock_guard<std::mutex> lk(mutex_);
	test_.push_back(t);
}

void traverse() {
	std::lock_guard<std::mutex> lk(mutex_);
	for (std::vector<Test>::const_iterator it = test_.begin(); it != test_.end(); ++it) {
		it->doInit();
	}
}
```

如果在doInit这里调用了post()，如果锁不可重入，可以立即死锁暴露问题。但如果是可重入，post()里做了push_back，改变了容器，可能导致迭代器失效而crash。