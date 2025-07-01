---
date: '2025-07-01T00:23:34+08:00'
draft: false
title: '2025 07 02 鸟群行为模拟'
---

# 涌现式群体行为模拟

link: [【游戏开发秘籍】用算法让NPC集体“开窍”？Boids鸟群算法详解！](https://www.bilibili.com/video/BV1XgM1zQEF5/)

鱼群、人群、蜂群等的自然模拟，使用Boids算法。

## 算法逻辑

$$
\vec{V} = \Pi c_i \vec{v_i} 
$$

其中，\(\vec{V}\)为最终向量，\(c_i\)为可调权重，\(\vec{v_i}\)为单规则向量。

我们有以下3条规则：

### 规则一：Cohesion, 向群体中心靠拢

目标：保持群体集聚。

行为：计算特定范围（通常为**视野范围**）内**同伴**的**平均位置**，然后产生一个朝向这个平均位置的驱动力。

### 规则二：Separation, 避免与同伴碰撞

目标：避免碰撞。

行为：对于每一个过近的同伴，产生一个排斥力，将这些排斥力加总。

### 规则三：Alignment, 与同伴移动方向保持一致。

目标：与群体行动方向趋同。

行为：计算特定范围（通常为**视野范围**）内**同伴**的**平均方向**，然后调整方向来匹配平均方向。

## 算法实现

```python
import numpy as np
import random
import matplotlib.pyplot as plt
import math
import statistics
import pygame

MAX_SPEED = 10.0
WEIGHT_COHESION = 1
WEIGHT_SEPARATION = 1
WEIGHT_ALIGNMENT = 1
WEIGHT_FORCE_AGAINST_WALL = 0
SOCIAL_DISTANCE = 20.0
FRAMERATE = 30

GRAY = (127, 127, 127)
WHITE = (255, 255, 255)

def cart2pol(x, y):
    rho = np.sqrt(x**2 + y**2)
    phi = math.degrees(np.arctan2(y, x))
    return((phi + 360) % 360, rho)

def pol2cart(phi, rho):
    x = rho * np.cos(math.radians(phi))
    y = rho * np.sin(math.radians(phi))
    return(x, y)

class EmergenceBehaviorDemo:
    class Unit:
        x = 0.0
        y = 0.0
        phi: float
        v = 0.0
        i: int
        def __init__(self, i):
            self.i = i
    
    class Playground:
        w = 1280.0
        h = 720.0
        def __init__(self):
            pass
    
    unit_index: int
    peasants: list[Unit]
    playground: Playground
    leader_id: int
    leader_walk_weight_of_previous_tick: float = 0.6
    plot_x: list[float]
    plot_y: list[float]
    
    def init_peasants(self, group_grid_width: int, playground: Playground, interval: float) -> list[Unit]:
        # Generate at the center of the playground
        
        class Center:
            x = playground.w/2
            y = playground.h/2
        
        class StartingCoord:
            x = Center.x - ((group_grid_width - 1) * interval / 2)
            y = Center.y - ((group_grid_width - 1) * interval / 2)
        
        self.unit_index = 0
        peasants = []
        for i in range(group_grid_width):
            for j in range(group_grid_width):
                new_unit = self.Unit(self.unit_index)
                new_unit.x = StartingCoord.x + (interval * i)
                new_unit.y = StartingCoord.y + (interval * j)
                new_unit.phi = random.random() * 360
                peasants.append(new_unit)
                self.unit_index += 1
        
        return peasants
    
    @classmethod            
    def print_peasants_coordinates(cls, l:list[Unit]):
        for unit in l:
            print(f"Peasant {unit.i}: ({unit.x}, {unit.y}), v = {unit.v}")
        return
    
    @classmethod  
    def print_peasant_coordinates(cls, u:Unit):
        print(f"Peasant {u.i}: ({u.x}, {u.y}), phi = {u.phi}° v = {u.v}")
        return
    
    @classmethod  
    def update_position(cls, u: Unit, p: Playground):
        u.x += u.v * math.cos(math.radians(u.phi)) 
        u.y += u.v * math.sin(math.radians(u.phi))
        u.x = u.x % p.w
        u.y = u.y % p.h
    
    @classmethod  
    def point_in_triangle(cls, p, t_1, t_2, t_3):
        # https://stackoverflow.com/questions/2049582/how-to-determine-if-a-point-is-in-a-2d-triangle
        def sign(p_1, p_2, p_3):
            return (p_1[0] - p_3[0]) * (p_2[1] - p_3[1]) - (p_2[0] - p_3[0]) * (p_1[1] - p_3[1])

        d1 = sign(p, t_1, t_2)
        d2 = sign(p, t_2, t_3)
        d3 = sign(p, t_3, t_1)
        
        has_neg = (d1 < 0) or (d2 < 0) or (d3 < 0)
        has_pos = (d1 > 0) or (d2 > 0) or (d3 > 0)
    
        return not (has_neg and has_pos)
    
    @classmethod  
    def get_distance(cls, p1, p2):
        return math.sqrt((p1[0] - p2[0])**2 + (p1[1] - p2[1])**2)
    
    def get_all_unit_in_eye_sight(self, u: Unit, fov: float = 90.0, social_distance: float = 20.0, view_distance: float = 300.0) -> list[Unit]:
        res = []
        vertex_1 = (u.x, u.y)
        vertex_2 = (u.x + math.cos(math.radians(u.phi+(fov / 2))) * view_distance, u.y + math.sin(math.radians(u.phi+(fov / 2))) * view_distance)
        vertex_3 = (u.x + math.cos(math.radians(u.phi-(fov / 2))) * view_distance, u.y + math.sin(math.radians(u.phi-(fov / 2))) * view_distance)
        
        # Now we have one triangle
        for peasant in self.peasants:
            if peasant.i == u.i:
                continue
            if self.point_in_triangle((peasant.x, peasant.y), vertex_1, vertex_2, vertex_3) or self.get_distance((u.x, u.y), (peasant.x, peasant.y)) <= social_distance:
                res.append(peasant)
        
        return res
    
    def get_all_unit_nearby(self, u: Unit, fov: float = 90.0, social_distance: float = 50.0, view_distance: float = 300.0) -> list[Unit]:
        res = []
        
        # Now we have one triangle
        for peasant in self.peasants:
            if peasant.i == u.i:
                continue
            if self.get_distance((u.x, u.y), (peasant.x, peasant.y)) <= social_distance:
                res.append(peasant)
        
        return res
    
    @classmethod  
    def combine_force_list(cls, l_force: list, l_weight: list = []):
        final_force_cart = [0, 0]
        
        if len(l_weight) == 0:
            actual_weight = [1] * len(l_force)
        else:
            actual_weight = l_weight
        
        for i in range(len(l_force)):
            (x, y) = pol2cart(l_force[i][0], l_force[i][1])
            final_force_cart[0] += x * actual_weight[i]
            final_force_cart[1] += y * actual_weight[i]
        
        final_force_rad = cart2pol(final_force_cart[0], final_force_cart[1])
        
        return final_force_rad
    
    @classmethod  
    def distance_between_point_and_line(cls, _lv1, _lv2, _p):
        lv1 = np.array(_lv1)
        lv2 = np.array(_lv2)
        p = np.array(_p)
        return np.abs(np.cross(lv2-lv1, lv1-p)) / np.linalg.norm(lv2-lv1)
    
    @classmethod  
    def force_against_the_walls(cls, u: Unit, p: Playground):
        v1 = (0.0, 0.0)
        v2 = (0.0, p.h)
        v3 = (p.w, 0.0)
        v4 = (p.w, p.h)
        s = (u.x, u.y)
        c = 200.0
        
        # Wall 1 v1-v2
        d1 = cls.distance_between_point_and_line(v1, v2, s)
        f1 = (90.0, c / d1)
        # Wall 2 v1-v3
        d2 = cls.distance_between_point_and_line(v1, v3, s)
        f2 = (180.0, c / d2)
        # Wall 3 v4-v2
        d3 = cls.distance_between_point_and_line(v4, v2, s)
        f3 = (0.0, c / d3)
        # Wall 4 v4-v3
        d4 = cls.distance_between_point_and_line(v4, v3, s)
        f4 = (270.0, c / d4)
        
        force_against_the_walls_list = [f1, f2, f3, f4]
        force_against_the_walls = cls.combine_force_list(force_against_the_walls_list)
        
        return force_against_the_walls
    
    def find_peasant_with_id(self, i):
        for u in self.peasants:
            if u.i == i: return u
        raise Exception('no such id')
    
    def update_behavior(
        self, 
        cohesion_weight: float = WEIGHT_COHESION, 
        separation_weight: float = WEIGHT_SEPARATION,
        social_distance: float = SOCIAL_DISTANCE, 
        alignment_weight: float = WEIGHT_ALIGNMENT,
        force_against_wall_weight: float = WEIGHT_FORCE_AGAINST_WALL
    ):
        
        new_direction_list = []
        for peasant in self.peasants:
            
            peasants_in_view = self.get_all_unit_in_eye_sight(peasant)
            peasants_nearby = self.get_all_unit_nearby(peasant)
            
            # Calculate cohesion
            if len(peasants_nearby) == 0:
                cohesion_force = (0, 0)
            else:
                
                average_position_in_view = (
                    statistics.mean(map(lambda x: x.x, peasants_nearby)),
                    statistics.mean(map(lambda x: x.y, peasants_nearby))
                )
                (xs, ys) = pol2cart(peasant.phi, peasant.v)
                cohesion_force = cart2pol((average_position_in_view[0]-xs), (average_position_in_view[1]-ys))
                
            # Calculate Separation
            separation_force_list = []
            for u in self.peasants:
                if u.i == peasant.i:
                    continue
                if self.get_distance((u.x, u.y), (peasant.x, peasant.y)) <= social_distance:
                    c = -1 / math.sqrt((u.y - peasant.y)**2 + (u.x - peasant.x)**2)
                    (x, y) = ((u.x - peasant.x) * c, (u.y - peasant.y) * c)
                    new_force = cart2pol(x, y)
                    separation_force_list.append(new_force)
            
            if len(separation_force_list) == 0:
                separation_force = (0, 0)
            else:
                separation_force = self.combine_force_list(separation_force_list)
            
            # Calculate Alignment
            if len(peasants_in_view) == 0:
                alignment_force = (0, 0)
            else:
                average_movement_in_view = (
                    statistics.mean(map(lambda x: x.phi, peasants_in_view)),
                    statistics.mean(map(lambda x: x.v, peasants_in_view))
                )
                
                (xt, yt) = pol2cart(average_movement_in_view[0], average_movement_in_view[1])
                (xs, ys) = pol2cart(peasant.phi, peasant.v)
                
                alignment_force = cart2pol((xt-xs), (yt-ys))
            
            # Calculate force against the wall
            force_against_wall = self.force_against_the_walls(peasant, self.playground)
            
            cohesion_force = (cohesion_force[0], cohesion_force[1]**0.75)
            alignment_force = (alignment_force[0], math.sqrt(alignment_force[1]))
            
            total_force = self.combine_force_list(
                [cohesion_force, separation_force, alignment_force, force_against_wall],
                [cohesion_weight, separation_weight, alignment_weight, force_against_wall_weight]
            )
            
            if peasant.i == self.leader_id:
                print(f"c_force:({cohesion_force[0]:.2f}°, {cohesion_force[1]:.2f}m/s) s_force:({separation_force[0]:.2f}°, {separation_force[1]:.2f}m/s) a_force:({alignment_force[0]:.2f}°, {alignment_force[1]:.2f}m/s) f_force:({force_against_wall[0]:.2f}°, {force_against_wall[1]:.2f}m/s)")
            
            new_direction_list.append((peasant.i, total_force))
            
        for (_i, (_phi, _v)) in new_direction_list:
            current_peasant = self.peasants[_i]
            current_peasant.phi = _phi
            current_peasant.v = min(_v, MAX_SPEED)
        
        for peasant in self.peasants:
            self.update_position(peasant, self.playground)
    
    def __init__(self, group_grid_width: int, interval: float):
        self.playground = self.Playground()
        self.peasants = self.init_peasants(group_grid_width, self.playground, interval)
        self.leader_id = random.randint(0, len(self.peasants)-1)
        self.plot_x = []
        self.plot_y = []
        pass

def pygame_rot_center(image, angle, x, y):
    
    rotated_image = pygame.transform.rotate(image, angle)
    new_rect = rotated_image.get_rect(center = image.get_rect(center = (x, y)).center)

    return rotated_image, new_rect

def main():
    run = True
    
    game = EmergenceBehaviorDemo(7, 10.0)
    
    pygame.init()
    window = pygame.display.set_mode((game.playground.w, game.playground.h))
    clock = pygame.time.Clock()
    
    pygame_image_fish = pygame.image.load('fish.png').convert_alpha()
    
    
    while run:
        clock.tick(FRAMERATE)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False
        
        game.update_behavior()

        window.fill(GRAY)
        for peasant in game.peasants:
            if peasant.i == game.leader_id:
                pygame.draw.rect(window, WHITE, (peasant.x - 5, peasant.y + 5, 10, 10))
            else:
                rotated_image, new_rect = pygame_rot_center(pygame_image_fish, (peasant.phi + 270) % 360, peasant.x - 5, peasant.y + 5)
                window.blit(rotated_image, new_rect)
        pygame.display.flip()
    
    plt.plot(game.plot_x, game.plot_y, color='blue')
    plt.grid(True)
    plt.savefig('foo.png')
    
    pygame.quit()

main()

```