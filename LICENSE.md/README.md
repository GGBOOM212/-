# -
线程池
package hello;
public class hello {
    public interface ThreadPool<Job extends Runnable>{
        void execute(Job job);
        void shutdown();
        void addWorkers(int num);
        void removeWorker(int num);
        void getJobSize();
    }
    public class DefaultThreadPool<Job extends Runnable> implements ThreadPool<Job>{
        private static final int MAX_WORKER_NUMBERS = 10;
        private static final int DEFAULT_WORKER_NUMBERS = 5;
        private static final int MIN_WORKER_NUMBERS = 1;
        private final LinkedList <Job> jobs = new LinkedList<Job>();
        private final List<Worker> workers = Collections.synchronizedList(new ArrayList<Worker>());
        private int workerNum;
        private AtomicLong  threadNum = new AtomicLong();
        public DefaultThreadPool() {
            this.workerNum = DEFAULT_WORKER_NUMBERS;
            initializeWorkers(this.workerNum);
        }

        public DefaultThreadPool(int num) {
            if (num > MAX_WORKER_NUMBERS) {
                this.workerNum =DEFAULT_WORKER_NUMBERS;
            } else {
                this.workerNum = num;
            }
            initializeWorkers(this.workerNum);
        }
        private void initializeWorkers(int num) {
            for (int i = 0; i < num; i++) {
                Worker worker = new Worker();
                workers.add(worker);
                Thread thread = new Thread(worker);
                thread.start();
            }
        }
        public void execute(Job job) {
            if (job != null) {
                synchronized (jobs) {
                    jobs.addLast(job);
                    jobs.notify();
                }
            }
        }
        public void shutdown() {
            for (Worker w : workers) {
                w.shutdown();
            }
        }
        public void addWorkers(int num) {
            synchronized (jobs) {
                if (num + this.workerNum > MAX_WORKER_NUMBERS) {
                    num = MAX_WORKER_NUMBERS - this.workerNum;
                }
                initializeWorkers(num);
                this.workerNum += num;
            }
        }
        public void removeWorker(int num) {
            synchronized (jobs) {
                if(num>=this.workerNum){
                    throw new IllegalArgumentException("超过了已有的线程数量");
                }
                for (int i = 0; i < num; i++) {
                    Worker worker = workers.get(i);
                    if (worker != null) {
                        worker.shutdown();
                        workers.remove(i);
                    }
                }
                this.workerNum -= num;
            }
        }
        public int getJobSize() {
            // TODO Auto-generated method stub
            return workers.size();
        }
        class Worker implements Runnable {
            private volatile boolean running = true;

            public void run() {
                while (running) {
                    Job job = null;
                    synchronized (jobs) {
                        if (jobs.isEmpty()) {
                            try {
                                jobs.wait();
                            } catch (InterruptedException e) {
                                Thread.currentThread().interrupt();
                                return;
                            }
                        }
                        job = jobs.removeFirst();
                    }
                    if (job != null) {
                        job.run();
                    }
                }
            }
            public void shutdown() {
                running = false;
            }
        }
    }
    }
