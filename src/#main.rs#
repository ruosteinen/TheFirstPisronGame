extern crate piston_window;

use piston_window::*;
use std::collections::{HashMap, HashSet};

// Компоненты
#[derive(Debug)]
struct Position {
    x: f64,
    y: f64,
}

struct Velocity {
    velocity_x: f64,
    velocity_y: f64,
}

struct Renderable {
    shape: Shape,
    color: [f32; 4],
}

enum Shape {
    Circle(f64), // Радиус
    Square(f64), // Размер стороны
}


// Сущность
#[derive(Debug, Clone, Copy, Hash, Eq, PartialEq)]
struct Entity(u32);


// Системы
struct MovementSystem;
struct RenderSystem;

impl MovementSystem {
    fn update(&self, window: &PistonWindow, positions: &mut HashMap<Entity, Position>, velocities: &mut HashMap<Entity, Velocity>, keys: &HashSet<Key>,player_entity: Entity,moving_square: Entity) {
        let window_size = window.size();
        for (entity, velocity) in velocities.iter_mut() {
            if let Some(position) = positions.get_mut(entity) {

                if *entity != player_entity {
                     position.x += velocity.velocity_x;
                     position.y += velocity.velocity_y;

                 }

                if *entity == moving_square{
                       
                    if position.x > window_size.width {
                        position.x = 0.0
                    } 
                    
                    if position.x < 0.0 {
                        position.x = window_size.width                  
                     }
                }
                                    
                 // Управление квадратом с помощью клавиш
                if *entity == player_entity{                                    
                    if keys.contains(&Key::W) { position.y -= velocity.velocity_y; }
                    if keys.contains(&Key::S) { position.y += velocity.velocity_y; }
                    if keys.contains(&Key::A) { position.x -= velocity.velocity_x; }
                    if keys.contains(&Key::D) { position.x += velocity.velocity_x; }
                 }
            }
        }
    }
}

impl RenderSystem {
    fn render(&self, c: Context, g: &mut G2d, positions: &HashMap<Entity, Position>, renderables: &HashMap<Entity, Renderable>) {
        for (entity, position) in positions.iter() {
            if let Some(renderable) = renderables.get(entity) {
                match renderable.shape {
                    Shape::Circle(radius) => {
                        ellipse(
                            renderable.color,
                            [position.x - radius, position.y - radius, radius * 2.0, radius * 2.0],
                            c.transform, g,
                        );
                    },
                    Shape::Square(size) => {
                        rectangle(
                            renderable.color,
                            [position.x, position.y, size, size],
                            c.transform, g,
                        );
                    },
                }
            }
        }
    }
}

fn main() {
    let mut window: PistonWindow = WindowSettings::new("ECS Example", [500, 500])
        .exit_on_esc(true).build().unwrap();
  
    let mut keys_pressed = HashSet::new();
    let mut next_entity_id = 0;
    let mut positions = HashMap::new();
    let mut velocities = HashMap::new();
    let mut renderables = HashMap::new();

    // Создание управляемого квадрата
    let player_entity = Entity(next_entity_id);
    next_entity_id += 1;
    positions.insert(player_entity, Position { x: 50.0, y: 50.0 });
    velocities.insert(player_entity, Velocity { velocity_x: 1.0, velocity_y: 1.0 });
    renderables.insert(player_entity, Renderable { shape: Shape::Square(50.0), color: [1.0, 0.0, 0.0, 1.0] }); // Красный квадрат
    // Добавляем статический квадрат для контраста
    
    let moving_square = Entity(next_entity_id);
    next_entity_id += 1;
    positions.insert(moving_square, Position { x: 200.0, y: 100.0 });
    velocities.insert(moving_square, Velocity { velocity_x: 0.09, velocity_y: 0.0 }); // Начальная скорость
    renderables.insert(moving_square, Renderable { shape: Shape::Square(100.0), color: [0.0, 0.0, 1.0, 1.0] }); // Синий квадрат    
    
    let mut last_cursor_position: Option<[f64; 2]> = None;
    
    while let Some(event) = window.next() {

               
         if let Some(cursor_position) = event.mouse_cursor_args() {
            last_cursor_position = Some(cursor_position);
        }
        
                
        if let Some(Button::Keyboard(key)) = event.press_args() {
            keys_pressed.insert(key);
        }
        
       
         if let Some(Button::Keyboard(key)) = event.release_args() {
            keys_pressed.remove(&key);
        }
        

        if let Some(Button::Mouse(button)) = event.press_args() {
            if button == MouseButton::Left {
                   
                    
            if let Some(cursor_position) = last_cursor_position {
                    
                    //println!("Позиция курсора: {:?}", cursor_position);
                                  
                    //if let Some(cursor_position) = event.mouse_cursor_args() {
                   // println!("Позиция курсора: {:?}", cursor_position);
                    let entity = Entity(next_entity_id);
                    next_entity_id += 1;

                    // Получение позиции игрока
                    let player_position = positions.get(&player_entity).unwrap();

                    // Вычисление вектора направления и скорости пули
                    let direction_x = cursor_position[0] - player_position.x;
                    let direction_y = cursor_position[1] - player_position.y;
                    let magnitude = (direction_x.powi(2) + direction_y.powi(2)).sqrt();
                    
                                                     
                    //println!("{:?}",magnitude);
                    /*let _bullet_velocity = Velocity {
                        velocity_x: 0.5 * direction_x / magnitude,
                        velocity_y: 0.5 * direction_y / magnitude
                    };*/

                    // Установка начальной позиции и скорости пули
                   
                    //let _bullet_position = Position { x: player_position.x, y: player_position.y };
                    
                    positions.insert(entity,Position { x: player_position.x, y: player_position.y });
                    
                    velocities.insert(entity,Velocity {velocity_x: 1.0 * direction_x / magnitude,velocity_y: 1.0 * direction_y / magnitude});
                    
                    renderables.insert(entity, Renderable { shape: Shape::Circle(5.0), color: [0.0, 1.0, 0.0, 1.0] }); // Зелёные пули
                }
            }
        }
      
        // Система движения
        let movement_system = MovementSystem;
        movement_system.update(&window,&mut positions, &mut velocities, &keys_pressed,player_entity,moving_square);

        window.draw_2d(&event, |c, g, _| {
            clear([1.0; 4], g);

            // Система рендеринга
            let render_system = RenderSystem;
            render_system.render(c, g, &positions, &renderables);
        });
    }
}
