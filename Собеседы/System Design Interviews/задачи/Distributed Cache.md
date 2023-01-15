# Distributed cache
Из этого курса https://www.youtube.com/watch?v=iuqZvajTOyA&ab_channel=SystemDesignInterview

## Требования
### функциональные требования
 - put(key, value)
 - get(key)

### нефункциональные требования
 - масштабируемость
 - высокая доступность
 - высокая производительность

## Компоненты


## Детальная проработка отдельных частей
### Алгоритм работы LRU-cache
![[LRU cache algorithm.png]]

Примерный код
```
class Node:
	def __init__(self, key: str, value: str):
		self.key = key
		self.value = value

		self.prev = self.next = None
	
class LRUCache:
	def __init__(self, capacity: int):
		self.map = {}
		self.capacity = capacity

		self.head = self.tail = None

	def get(self, key: str) -> Optional[str]:
		if not (node := self.map.get(key)):
			return None
		
		self._delete_from_list(node)
		self._set_head(node)
		
		return node.value
	
	def put(self, key: str, value: str):
		if node := self.map.get(key):
			node.value = value

			self._delete_from_list(node)
			self._set_head(node)
			return

		if len(self.map) >= self.capacity:
			del self.map[self.tail.key]
			self._delete_from_list(self.tail)
		
		node = Node(key, value)
		map[key] = node
		self._set_head(node)
	
	def _set_head(self, node: Node):
		self.head.next, self.head, node.prev = node, node, self.head
	
	def _delete_from_list(self, node: Node):
		if node.prev:
			npde.prev.next = node.next
		if node.next:
			node.next.prev = node.prev
			
```

