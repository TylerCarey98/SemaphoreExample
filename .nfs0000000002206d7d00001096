#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <semaphore.h>
#include <mutex>
#include <list>

using namespace std;

void *process1 (void* dummy);
void *process2 (void* dummy);
void *process3 (void* dummy);

void waitFreeOne();

int n = 2;
sem_t freefull, onefull, twofull, p1, p2;
list<int> freelist(n, 0), list1(0), list2(0);
mutex mtx, freelock, lock1, lock2;

int main () {
	sem_init(&freefull, 0, n);
	sem_init(&onefull, 0, 0);
	sem_init(&twofull, 0, 0);
	sem_init(&p1, 0, 1);
	sem_init(&p2, 0, 0);
	pthread_t t1, t2, t3;
	pthread_create(&t1, NULL, process1, NULL);
	pthread_create(&t2, NULL, process2, NULL);
	pthread_create(&t3, NULL, process3, NULL);
	pthread_join(t1, NULL);
	pthread_join(t2, NULL);
	pthread_join(t3, NULL);
}

void *process1 (void* dummy) {
	int b;
	while (1) {
		sem_wait(&p1);
		mtx.lock();
		if (freelist.size() < 1) {
			mtx.unlock();
			sem_wait(&freefull);
		} else {
			sem_wait(&freefull);
			mtx.unlock();
		}

		freelock.lock();
		b = freelist.front();
		freelist.pop_front();
		freelock.unlock();

		b = 2;
		lock1.lock();
		list1.push_back(b);
		lock1.unlock();
		sem_post(&onefull);
		sem_post(&p2);
	}
}

void *process2 (void* dummy) {
	int x, y;
	while (1) {
		sem_wait(&p2);

		waitFreeOne();

		lock1.lock();
		x = list1.front();
		list1.pop_front();
		lock1.unlock();

		freelock.lock();
		y = freelist.front();
		freelist.pop_front();
		freelist.push_back(x);
		freelock.unlock();
		sem_post(&freefull);

		y = x*5;

		lock2.lock();
		list2.push_back(y);
		lock2.unlock();
		sem_post(&twofull);
	}
}

void *process3 (void* dummy) {
	int c;
	while (1) {
		sem_wait(&twofull);
		lock2.lock();
		c = list2.front();
		list2.pop_front();
		lock2.unlock();

		printf("%d\n", c);
		fflush(stdout);

		freelock.lock();//pthread_mutex_lock(&lockfree);
		freelist.push_back(c);
		freelock.unlock();//pthread_mutex_unlock(&lockfree);
		sem_post(&freefull);
		sem_post(&p1);
	}
}

void waitFreeOne() {
	mtx.lock();
	while (1) {
		if (freelist.size() < 1) {
			sem_wait(&freefull);
			continue;
		}

		if (list1.size() < 1) {
			sem_wait(&onefull);
			continue;
		}

		break;
	}

	sem_wait(&freefull);
	sem_wait(&onefull);
	mtx.unlock();
}
