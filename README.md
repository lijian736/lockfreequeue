# lockfreequeue

#### 介绍
使用C++11原子操作实现的高性能自旋锁队列

#### 软件架构
软件架构说明


#### 安装教程

直接使用

#### 使用说明

```
#include <thread>
#include <atomic>
#include <string>
#include <iostream>
#include <sstream>
#include <chrono>
#include <stdlib.h>
#include "common_lockfree_queue.h"

/**
* the item in the lock-free queue
*/
class TestEntity
{
public:
	TestEntity(int id_p = 0, int value_p = 0)
	{
		this->id = id_p;
		this->value = value_p;
	}

	void display()
	{
		std::cout << "id = " << id << " value = " << value << std::endl;
	}

private:
	int id;
	int value;
};

LockFreeQueue<TestEntity> queue(4096);

#define COUNT (1000 * 1000)

void produce()
{
	auto beginTime = std::chrono::high_resolution_clock::now();
	unsigned int i = 0;
	while (i < COUNT)
	{
		if (queue.push(TestEntity(i, i)))
		{
			i++;
		}
	}
	auto endTime = std::chrono::high_resolution_clock::now();
	auto elapsedTime = std::chrono::duration_cast<std::chrono::milliseconds>(endTime - beginTime);

	std::cout << "Producer thread id:[" << std::this_thread::get_id() << "], IO:[" << COUNT * sizeof(TestEntity) * 1.0 / (elapsedTime.count() * 1024 * 1024) * 1000
		<< " MB/s], messages:[" << COUNT * 1.0 / elapsedTime.count() * 1000 << " /s], elapsed:[" << elapsedTime.count()*1.0 / 1000 << "s], item count:[" << COUNT << "]" << std::endl;
}


std::atomic<unsigned int> total_consume(0);

void consume()
{
	TestEntity test;
	auto beginTime = std::chrono::high_resolution_clock::now();
	unsigned int i = 0;
	while (total_consume < 2 * COUNT)
	{
		if (queue.pop(test))
		{
		//	test.display();
			i++;
			total_consume++;
		}
	}
	auto endTime = std::chrono::high_resolution_clock::now();
	auto elapsedTime = std::chrono::duration_cast<std::chrono::milliseconds>(endTime - beginTime);

	std::cout << "Consumer thread id:[" << std::this_thread::get_id() << "], IO:[" << i * sizeof(TestEntity) * 1.0 / (elapsedTime.count() * 1024 * 1024) * 1000
		<< " MB/s], messages:[" << i * 1.0 / elapsedTime.count() * 1000 << " /s], elapsed:[" << elapsedTime.count()*1.0 / 1000 << "s], item count:[" << i << "]" << std::endl;
}

int main(int argc, char const *argv[])
{
	(void)argc;
	(void)argv;

	std::thread producer1(produce);
	std::thread producer2(produce);
	std::thread consumer1(consume);
	std::thread consumer2(consume);

	producer1.join();
	producer2.join();
	consumer1.join();
	consumer2.join();

	std::cin.get();

	return 0;
}

```
