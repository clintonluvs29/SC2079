## SC2079 MDP (Algorithm team)
main/main in c - main code for image recognition task
fastest car/fastest car in c - main code for fastest car task

change segments_to_commands: code to convert cmds into "LEFT", degrees format instead of the old "L", arc length format

### further improvements (to be made and stored in 'main edited'): 
In: def full_dubins_commands(path, goals, R):
Replace: path_type, segments = dubins_shortest_path(q0, q1, R)
With:
obstacles = build_obstacles(
    [goals[k] for k in range(len(goals)) if k not in (path[i], path[i+1])]
)
edge = dubins_path_valid(q0, q1, R, obstacles)
if edge is None:
    raise RuntimeError(f"No valid Dubins path between {q0} and {q1}")
path_type, segments, _ = edge
#### Do this in both:
full_dubins_commands
flattened_dubins_commands
#### ----
before TSP: edge_cache = {}
def get_edge(i, j, goals, R, edge_cache):
    if (i, j) in edge_cache:
        return edge_cache[(i, j)]

    obstacles = build_obstacles(
        [goals[k] for k in range(len(goals)) if k not in (i, j)]
    )

    result = dubins_path_valid(goals[i], goals[j], R, obstacles)

    edge_cache[(i, j)] = result
    return result
Then replace edge = dubins_path_valid(points[v], points[w], R, obstacles) with edge = get_edge(v, w, points, R, edge_cache)
Do this in: nearest_neighbour_optimized, path_length, improve_path
#### -----
Iterative instead of one-off:
Instead of flattened_dubins_commands we create next_commands(current_pose, remaining_goals, R)

def goal_reached(current_pose, goal, pos_tol=2.0, angle_tol_deg=10):
    dx = current_pose[0] - goal[0]
    dy = current_pose[1] - goal[1]
    dist = math.hypot(dx, dy)
    angle_error = abs(
        math.degrees(mod2pi(current_pose[2] - goal[2]))
    )
    return dist < pos_tol and angle_error < angle_tol_deg
    
def iterative_dubins_step(current_pose, remaining_goals, R, edge_cache):
    """
    Returns commands to next goal.
    Removes goal if reached.
    """
    if not remaining_goals:
        return None, remaining_goals
    # Check if already reached first goal
    if goal_reached(current_pose, remaining_goals[0]):
        print("Goal reached. Removing it.")
        remaining_goals.pop(0)
        if not remaining_goals:
            return None, remaining_goals
    next_goal = remaining_goals[0]
    # Build obstacles from remaining goals (except target)
    obstacles = build_obstacles(
        [g for g in remaining_goals if g != next_goal]
    )
    edge = dubins_path_valid(current_pose, next_goal, R, obstacles)
    if edge is None:
        raise RuntimeError("No valid Dubins path from current pose")
    path_type, segments, _ = edge
    commands = segments_to_commands(path_type, segments, R)
    return commands, remaining_goals

remaining_goals = [goals[i] for i in path]
while remaining_goals:
    current_pose = get_robot_pose()   # from localization
    commands, remaining_goals = iterative_dubins_step(
        current_pose,
        remaining_goals,
        radius,
        edge_cache
    )
    if commands is None:
        break
    send_commands_to_robot(commands)
