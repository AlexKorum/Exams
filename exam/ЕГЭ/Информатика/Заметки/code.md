```python
from functools import wraps

def track_game_tree(func):
    """Декоратор, который строит Mermaid-дерево по уровням (слева направо)."""
    graph_data = []
    visited = set()
    level_nodes = {}  # level -> set of nodes

    def mermaid_node(s, level, is_win=False):
        """Генерирует уникальный ID узла и его отображение."""
        node_id = f"s{level}_{s}"
        label = f"{s}{' (выигрыш)' if is_win else ''}"
        return node_id, label

    @wraps(func)
    def wrapper(s, m):
        # Если это конечное состояние — добавляем узел
        if s >= 125:
            node_id, label = mermaid_node(s, 0, is_win=True)
            if node_id not in visited:
                graph_data.append(f'    {node_id}["{label}"]')
                visited.add(node_id)
                level_nodes.setdefault(0, set()).add(node_id)
            return func(s, m)

        # Генерируем узел текущего состояния
        node_id, label = mermaid_node(s, m)
        if node_id not in visited:
            graph_data.append(f'    {node_id}["{label}"]')
            visited.add(node_id)
            level_nodes.setdefault(m, set()).add(node_id)

        # Если глубина 0 — возвращаем
        if m == 0:
            return func(s, m)

        # Генерируем ходы
        moves = [
            (s + 2, "+2"),
            (s + 4, "+4"),
            (s * 2, "×2")
        ]

        for next_s, move in moves:
            next_node_id, _ = mermaid_node(next_s, m - 1, is_win=(next_s >= 125))

            # Добавляем узел следующего состояния, если ещё не добавлен
            if next_node_id not in visited:
                graph_data.append(f'    {next_node_id}["{next_s}{" (выигрыш)" if next_s >= 125 else ""}"]')
                visited.add(next_node_id)
                level_nodes.setdefault(m - 1, set()).add(next_node_id)

            # Добавляем ребро
            edge_label = move
            graph_data.append(f'    {node_id} --> |{edge_label}| {next_node_id}')

        return func(s, m)

    def get_mermaid_tree():
        """Возвращает Mermaid-код дерева (LR — слева направо)."""
        header = "```mermaid\ngraph TD\n"
        body = "\n".join(graph_data)
        footer = "\n```"
        return header + body + footer

    # Добавляем методы
    wrapper.get_tree = get_mermaid_tree
    wrapper.reset = lambda: graph_data.clear() or visited.clear() or level_nodes.clear()

    return wrapper

# --- Пример использования ---

@track_game_tree
def f(s, m):
    if s >= 125: return m % 2 == 0
    if m == 0: return 0
    h = [
        f(s + 2, m - 1),
        f(s + 4, m - 1),
        f(s * 2, m - 1),
    ]
    return any(h) if m % 2 != 0 else all(h)

# --- Запуск ---
def main():
    f.reset()
    result = f(61, 2)
    graph = f.get_tree()
    print(graph)

if __name__ == '__main__':
    main()
```
