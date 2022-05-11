```rust
use std::thread;
use std::sync::{Mutex,Arc,mpsc};

pub enum Message {
    NewTask(Task),
    Exit,
}

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

pub struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

type Task = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    pub fn new(cap: usize) -> Self {
        assert!(cap>0);
        let (sender,receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        let mut workers = Vec::with_capacity(cap);
        for i in 0..cap {
            workers.push(Worker::new(i,receiver.clone()));
        }
        ThreadPool{
            workers: workers,
            sender: sender,
        }
    }
    pub fn execute<F: FnOnce()+'static+Send>(&self, f: F) {
        self.sender.send(Message::NewTask(Box::new(f))).unwrap()
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        for _ in &self.workers {
            self.sender.send(Message::Exit).unwrap()
        }
        println!("all thread stopping......");
        println!("{}",self.workers.len());
        for worker in &mut self.workers {
            println!("thread {} was stop!",worker.id);
            if let Some(t) = worker.thread.take() {
                t.join().unwrap();
            }
        }
    }
}

impl Worker {
    fn new(id: usize,receiver: Arc<Mutex<mpsc::Receiver<Message>>>) -> Self {
        let t = thread::spawn(move || loop{
            let message = receiver.lock().unwrap().recv().unwrap();
            match message{
                Message::NewTask(f) => {
                    println!("threadId: {}",id);
                    f();
                },
                Message::Exit => {
                    println!("worker {} stop",id);
                    break;
                }
            }

        });
        Worker{
            id: id,
            thread: Some(t),
        }
    }
}

fn main() {}
```
