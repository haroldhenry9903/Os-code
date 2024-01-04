#include <iostream>
#include <vector>
#include <queue>

using namespace std;

struct Burst {
    int duration;
    bool isCPU; // Indicates whether the burst is a CPU burst (true) or an I/O burst (false)

    Burst(int dur, bool cpu) : duration(dur), isCPU(cpu) {}
};

struct Job {
    int id;
    int arrivalTime; // Arrival time for the job
    vector<Burst> bursts;

    Job(int jobId, int arrival, vector<pair<int, bool>>& burstDetails)
        : id(jobId), arrivalTime(arrival) {
        for (const auto& detail : burstDetails) {
            Burst burst(detail.first, detail.second);
            bursts.push_back(burst);
        }
    }
};

int main() {
    int quantum;
    int numJobs;

    // Assign quantum and the number of jobs (customize as needed)
    quantum = 11; // Example quantum value
    numJobs = 6; // Example number of jobs

    vector<Job> jobs;
    queue<Job*> readyQueue;
    vector<Job*> ioQueue;

    // Define job details without specifying arrival times
    vector<vector<pair<int, bool>>> jobDetails = {
        {{2, true}, {3, false}, {1, true}}, // Job 1: (duration, isCPU)
        {{1, true}, {4, false}, {2, true}},  // Job 2
        {{1, true}, {4, false}, {2, true}},  // Job 3
        {{1, true}, {4, false}, {2, true}},  // Job 4
        {{1, true}, {4, false}, {21, true}},  // Job 5
        {{11, true}, {41, false}, {21, true}}  // Job 6

    };

    // Specify arrival times for each job
    vector<int> arrivalTimes = {0, 3 ,4,5,6, 7}; // Arrival times for each job

    int jobId = 1;
    for (int i = 0; i < numJobs; i++) {
        Job job(jobId++, arrivalTimes[i], jobDetails[i]);
        jobs.push_back(job);
    }

    int currentTime = 0;
    int totalWaitingTime = 0;
    int totalTurnaroundTime = 0;
    int completedJobs = 0;
    int totalInterrupts = 0; // Total number of interrupts (context switches)

    while (completedJobs < numJobs) {
        // Add arrived jobs to the ready queue
        for (Job& job : jobs) {
            if (job.arrivalTime <= currentTime && job.bursts.size() > 0) {
                readyQueue.push(&job);
                totalInterrupts++; // Count an arrival interrupt
            }
        }

        if (!readyQueue.empty()) {
            Job* currentJob = readyQueue.front();
            readyQueue.pop();

            Burst& currentBurst = currentJob->bursts.front();
            int executionTime = min(quantum, currentBurst.duration);
            currentTime += executionTime;
            currentBurst.duration -= executionTime;

            if (currentBurst.duration == 0) {
                currentJob->bursts.erase(currentJob->bursts.begin());
                if (!currentBurst.isCPU && !currentJob->bursts.empty()) {
                    ioQueue.push_back(currentJob);
                    totalInterrupts++; // Count an I/O interrupt (context switch) when entering I/O queue
                } else if (currentJob->bursts.empty()) {
                    completedJobs++;
                    int turnaroundTime = currentTime - currentJob->arrivalTime; // Calculate turnaround time including arrival time
                    totalWaitingTime += turnaroundTime - currentJob->id; // Calculate waiting time
                    totalTurnaroundTime += turnaroundTime;
                    totalInterrupts++; // Count a job completion interrupt
                }
            } else {
                totalInterrupts++; // Count a CPU interrupt (context switch) due to quantum expiry
            }
        } else if (!ioQueue.empty()) {
            vector<Job*> newIoQueue;
            for (Job* job : ioQueue) {
                Burst& currentBurst = job->bursts.front();
                currentBurst.duration--;
                if (currentBurst.duration == 0) {
                    job->bursts.erase(job->bursts.begin());
                    readyQueue.push(job);
                    if (currentBurst.isCPU) {
                        totalInterrupts++; // Count a CPU interrupt (context switch) when I/O operation completes and job returns to ready queue
                    } else {
                        totalInterrupts++; // Count an I/O interrupt (context switch) when I/O operation completes and job returns to ready queue
                    }
                } else {
                    newIoQueue.push_back(job);
                }
            }
            ioQueue = newIoQueue;
        } else {
            currentTime++;
        }



    }

    // Calculate average turnaround time and average waiting time
    double averageTurnaroundTime = static_cast<double>(totalTurnaroundTime) / numJobs;
    double averageWaitingTime = static_cast<double>(totalWaitingTime) / numJobs;

    // Print results including the total interrupt count
    cout << "Quantum:" << quantum <<endl;
    cout << "Overall Waiting Time: " << totalWaitingTime << endl;
    cout << "Overall Turnaround Time: " << totalTurnaroundTime << endl;
    cout << "Total Number of Interrupts (Context Switches): " << totalInterrupts << endl;
    cout << "Average Waiting Time: " << averageWaitingTime << endl;
    cout << "Average Turnaround Time: " << averageTurnaroundTime << endl;

    return 0;
}
