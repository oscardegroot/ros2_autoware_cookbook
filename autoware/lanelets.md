# Lanelets
For working with the map, there is `route_handler` package that can read the map and a route, allowing for easy access of the roadmap.

An example (derived from `behavior_path_planner/planner_manager`) that I use:

```cpp
route_handler_->setMap(*map_ptr_);
route_handler_->setRoute(*route_ptr_);

/** @see behavior_path_planner/planner manager */
geometry_msgs::msg::Pose cur_pose = odometry_ptr_->pose.pose;
PathWithLaneId reference_path{}; // The final reference path

const auto backward_length = 10.; // How far back should the path go
const auto forward_length = 100.; // How far forward should the path go

reference_path.header = route_handler_->getRouteHeader();

// Find the closest lanelet to our lanelet
lanelet::ConstLanelet closest_lane{};
route_handler_->getClosestLaneletWithinRoute(cur_pose, &closest_lane);

const auto current_lanes = route_handler_->getLaneletSequence(
    closest_lane, cur_pose, backward_length, forward_length);

reference_path = route_handler_->getCenterLinePath(current_lanes, backward_length, forward_length, true);
```