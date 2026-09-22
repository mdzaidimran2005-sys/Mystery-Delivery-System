# Mystery-Delivery-System

import json
import math


# ---------------------------------------------------------
# 1. Read and parse data.json
# ---------------------------------------------------------

with open("data.json", "r") as file:
    data = json.load(file)


warehouses = data["warehouses"]
agents = data["agents"]
packages = data["packages"]


# ---------------------------------------------------------
# 2. Function to calculate Euclidean distance
# ---------------------------------------------------------

def distance(point1, point2):
    x1, y1 = point1
    x2, y2 = point2

    return math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2)


# ---------------------------------------------------------
# Store the current location of every agent
# ---------------------------------------------------------

agent_locations = {}

for agent_id, location in agents.items():
    agent_locations[agent_id] = location[:]


# Store results for every agent
report = {}

for agent_id in agents:
    report[agent_id] = {
        "packages_delivered": 0,
        "total_distance": 0.0
    }


# ---------------------------------------------------------
# 3. Assign each package to the nearest agent
# ---------------------------------------------------------

for package in packages:

    package_id = package["id"]
    warehouse_id = package["warehouse"]
    destination = package["destination"]

    warehouse_location = warehouses[warehouse_id]

    nearest_agent = None
    shortest_distance = float("inf")

    # Find nearest agent to the warehouse
    for agent_id, agent_location in agent_locations.items():

        d = distance(agent_location, warehouse_location)

        if d < shortest_distance:
            shortest_distance = d
            nearest_agent = agent_id

    # -----------------------------------------------------
    # Simulate delivery
    # Agent travels:
    # current location -> warehouse -> destination
    # -----------------------------------------------------

    current_location = agent_locations[nearest_agent]

    distance_to_warehouse = distance(
        current_location,
        warehouse_location
    )

    distance_to_destination = distance(
        warehouse_location,
        destination
    )

    total_package_distance = (
        distance_to_warehouse +
        distance_to_destination
    )

    # Update agent's total distance
    report[nearest_agent]["total_distance"] += total_package_distance

    # Update number of packages delivered
    report[nearest_agent]["packages_delivered"] += 1

    # Agent is now at the package destination
    agent_locations[nearest_agent] = destination

    print(
        package_id,
        "assigned to",
        nearest_agent,
        "| Distance:",
        round(total_package_distance, 2)
    )


# ---------------------------------------------------------
# 4. Calculate efficiency and prepare final report
# ---------------------------------------------------------

best_agent = None
best_efficiency = float("inf")

for agent_id in report:

    packages_delivered = report[agent_id]["packages_delivered"]
    total_distance = report[agent_id]["total_distance"]

    if packages_delivered > 0:
        efficiency = total_distance / packages_delivered
    else:
        efficiency = 0

    report[agent_id]["total_distance"] = round(total_distance, 2)
    report[agent_id]["efficiency"] = round(efficiency, 2)

    # Lower distance per package = greater efficiency
    if packages_delivered > 0 and efficiency < best_efficiency:
        best_efficiency = efficiency
        best_agent = agent_id


report["best_agent"] = best_agent


# ---------------------------------------------------------
# 5. Save report to report.json
# ---------------------------------------------------------

with open("report.json", "w") as file:
    json.dump(report, file, indent=4)


# Display final report
print("\nFinal Report:")
print(json.dumps(report, indent=4))

print("\nReport saved successfully to report.json")
