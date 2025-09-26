"""
Hasse diagram generator with minimal/maximal/greatest/least/lub/glb
Commit the output PNG/SVG to your GitHub repo.
"""

import networkx as nx
import matplotlib.pyplot as plt

# ---------- Define your poset ----------
# Example: set {1,2,3,6} under divisibility
elements = {1, 2, 3, 6}
relation = lambda a, b: b % a == 0   # "a divides b"

# Build full order graph (DAG)
G = nx.DiGraph()
for a in elements:
    for b in elements:
        if a != b and relation(a, b):
            G.add_edge(a, b)

# Transitive reduction → Hasse edges
H = nx.transitive_reduction(G)

# ---------- Compute properties ----------
# Minimal = no predecessors
minimal = [n for n in H.nodes if H.in_degree(n) == 0]
# Maximal = no successors
maximal = [n for n in H.nodes if H.out_degree(n) == 0]

# Least element = unique minimal that relates to all
least = [m for m in minimal if all(nx.has_path(H, m, x) or m == x for x in H.nodes)]
# Greatest element = unique maximal that relates to all
greatest = [M for M in maximal if all(nx.has_path(H, x, M) or x == M for x in H.nodes)]

# LUB (join) and GLB (meet) for a given subset
def lub(subset):
    """Least upper bound (join) of subset if exists"""
    uppers = [x for x in H.nodes if all(nx.has_path(H, s, x) or s == x for s in subset)]
    # minimal among uppers
    candidates = [u for u in uppers if not any((u != v and nx.has_path(H, v, u)) for v in uppers)]
    return candidates

def glb(subset):
    """Greatest lower bound (meet) of subset if exists"""
    lowers = [x for x in H.nodes if all(nx.has_path(H, x, s) or s == x for s in subset)]
    # maximal among lowers
    candidates = [l for l in lowers if not any((l != v and nx.has_path(H, l, v)) for v in lowers)]
    return candidates

# Example: join and meet of {2,3}
example_subset = [2,3]
print("LUB(2,3):", lub(example_subset))
print("GLB(2,3):", glb(example_subset))

print("Minimal elements:", minimal)
print("Maximal elements:", maximal)
print("Least element:", least)
print("Greatest element:", greatest)

# ---------- Draw diagram ----------
# Layout by topological levels
levels = {}
for node in nx.topological_sort(H):
    preds = list(H.predecessors(node))
    if not preds:
        levels[node] = 0
    else:
        levels[node] = max(levels[p] + 1 for p in preds)

pos = {}
by_level = {}
for n, lvl in levels.items():
    by_level.setdefault(lvl, []).append(n)

for lvl, nodes in by_level.items():
    for i, n in enumerate(sorted(nodes)):
        pos[n] = (i, -lvl)

# Node colors
color_map = []
for n in H.nodes:
    if n in least: color_map.append("lightblue")
    elif n in greatest: color_map.append("lightgreen")
    elif n in minimal: color_map.append("orange")
    elif n in maximal: color_map.append("pink")
    else: color_map.append("white")

plt.figure(figsize=(5,5))
nx.draw_networkx_nodes(H, pos, node_color=color_map, edgecolors='black')
nx.draw_networkx_labels(H, pos)
nx.draw_networkx_edges(H, pos, arrows=False, width=1.5)
plt.axis("off")
plt.tight_layout()
plt.savefig("hasse.png", dpi=200)
plt.savefig("hasse.svg")
print("Saved hasse.png and hasse.svg")
