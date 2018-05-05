# Samples of building Distributed TensorFlow Docker image

This project contains the docker file to build distributed tensorflow for both CPU and GPU.

## Distributed TensorFlow

The [Distributed TensorFlow](https://www.tensorflow.org/deploy/distributed)
documentation for a description of how it works. The examples in this
repository focus on the most common form of distributed training: between-graph
replication with asynchronous updates.

### Common Setup for distributed training

Every distributed training program has some common setup. First, define flags so
that the worker knows about other workers and knows what role it plays in
distributed training:

```python
# Flags for configuring the task
flags.DEFINE_integer("task_index", None,
                     "Worker task index, should be >= 0. task_index=0 is "
                     "the master worker task the performs the variable "
                     "initialization.")
flags.DEFINE_string("ps_hosts", None,
                    "Comma-separated list of hostname:port pairs")
flags.DEFINE_string("worker_hosts", None,
                    "Comma-separated list of hostname:port pairs")
flags.DEFINE_string("job_name", None, "job name: worker or ps")
```

Then, start your server. Since worker and parameter servers (ps jobs) usually
share a common program, parameter servers should stop at this point and so they
are joined with the server.

```python
# Construct the cluster and start the server
ps_spec = FLAGS.ps_hosts.split(",")
worker_spec = FLAGS.worker_hosts.split(",")

cluster = tf.train.ClusterSpec({
    "ps": ps_spec,
    "worker": worker_spec})

server = tf.train.Server(
    cluster, job_name=FLAGS.job_name, task_index=FLAGS.task_index)

if FLAGS.job_name == "ps":
  server.join()
```

Afterwards, the code varies depending on the form of distributed training you
intend on doing. The most common form is between-graph replication.

## Build docker image

```
make
```