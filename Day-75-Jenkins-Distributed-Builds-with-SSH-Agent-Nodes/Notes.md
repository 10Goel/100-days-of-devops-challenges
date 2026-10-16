# Day 75 — Jenkins Controller–Agent Architecture: Technical Notes

## 1. Why Jenkins uses agents

A Jenkins **controller** hosts configuration, jobs, scheduling, authorization, and orchestration. An **agent** is a worker that executes assigned build steps. Distributed agents isolate build environments, enable parallel builds, and allow jobs to target systems with specific tools or operating systems.

```text
Controller  --schedules-->  Agent with matching label
                              |
                              +-- executes build in workspace
                              +-- sends logs/results back
```

A node being **registered** in Jenkins does not mean it is **online** or able to run builds.

## 2. Core vocabulary

| Term | Meaning |
|---|---|
| Node | A machine or configured execution endpoint known to Jenkins |
| Permanent Agent | A manually managed worker node that remains configured |
| Executor | A slot for running a concurrent build on a node |
| Label | Tag used by the scheduler to select eligible agents |
| Remote root directory | Agent-side directory for remoting state, tools, and workspaces |
| SSH Build Agents | Jenkins plugin that connects to a remote host and launches Remoting using SSH |
| Remoting | Jenkins communication layer between controller and agent |
| `remoting.jar` | Java component started on the agent for controller–agent communication |

A node showing `0/1` executors busy means **zero of one slots currently occupied**; it does not by itself indicate a failed node.

## 3. SSH-based agent launch sequence

```text
1. Controller opens TCP connection to agent host:22
2. SSH server identity checked against controller's known_hosts
3. Remote user authenticated with Jenkins Credentials
4. Remote root directory checked/used
5. remoting.jar transferred or verified over SFTP
6. Java executable specified by Java Path starts remoting.jar
7. Remoting channel starts
8. Node becomes Online and can accept builds
```

The actual successful launch showed both `channel started` and `Agent successfully connected and online`.

## 4. Host authentication versus user authentication

**Host authentication** answers, “Is this really the expected SSH server?” It checks the server's public host key against `~/.ssh/known_hosts` on the **Jenkins controller Linux account**.

**User authentication** answers, “Can Jenkins log in as this remote account?” The SSH Build Agents plugin uses Jenkins Credentials (in this lab, separate username/password entries).

These are independent: accepting a known host does *not* eliminate the need for valid credentials. Similarly, successful user authentication does not guarantee the agent process can launch.

Never use `StrictHostKeyChecking=no` as an unexplained workaround. Host fingerprints should be verified through trusted lab or administrative information before accepting a new host key.

## 5. Java and class-file versions

Jenkins Remoting is Java bytecode and must run on a compatible JRE.

| Class-file major version | Java runtime |
|---|---|
| 55 | Java 11 |
| 61 | Java 17 |
| 65 | Java 21 |

Observed error:

```text
UnsupportedClassVersionError:
hudson/remoting/Launcher ... class file version 61.0
this version of the Java Runtime only recognizes ... up to 55.0
```

Interpretation: Remoting required at least Java 17 bytecode support, but the selected Java was 11. For this lab, Java 21 was installed and explicitly selected. Different Jenkins/Remoting releases have different minimum supported Java versions, so verify the supported pairing instead of assuming Java 17 is always sufficient.

## 6. Why installing Java 21 did not fix the failure automatically

Several JDK/JRE packages can coexist. `java` on `PATH` may continue pointing to Java 11 because of Linux alternatives. Installing Java 21 does **not** necessarily switch the default.

Safer options:

1. Configure Jenkins node **Advanced → Java Path** to an absolute Java 21 executable verified on that host.
2. Use a carefully verified stable symlink per user such as `/home/tony/java21`.

In this challenge, a hardcoded old RPM path caused:

```text
bash: ... /usr/lib/jvm/java-21-openjdk-21.0.12.../bin/java: No such file or directory
Agent JVM has terminated. Exit code=127
```

Exit `127` is the shell's command-not-found result. It is not an SSH credential error. The correction was to locate the actual Java executable in the current lab and use a tested symlink.

## 7. Symlinks and correctness

Example:

```bash
JAVA21=$(rpm -ql java-21-openjdk-headless | grep '/bin/java$' | head -n 1)
test -n "$JAVA21" && test -x "$JAVA21"
ln -sfn "$JAVA21" "$HOME/java21"
"$HOME/java21" -version
```

`ln -sfn` replaces the link path if it already exists; inspect the target before running this on non-lab machines. A link remains valid only as long as its target exists. Per-node Java Path is scoped to the Jenkins agent and avoids changing the system-wide runtime used by other applications.

## 8. Labels, scheduling, and distributed execution

Given labels `stapp01`, `stapp02`, `stapp03`, a job restricted to `stapp02` can execute on `App_server_2` if it is online and has an available executor.

A simple Declarative Pipeline illustration (not required to pass Day 75):

```groovy
pipeline {
    agent { label 'stapp02' }
    stages {
        stage('Verify host') {
            steps {
                sh '''
                    echo "Node: $NODE_NAME"
                    hostname
                    whoami
                '''
            }
        }
    }
}
```

This illustrates **label-based scheduling**. A label does not create a connection; the matching agent must first be available.

## 9. Common agent failures: troubleshooting matrix

| Symptom | Stage of failure | Next check |
|---|---|---|
| No `known_hosts` file | SSH host verification | Jenkins Linux account's `~/.ssh/known_hosts` |
| Host key mismatch | SSH host verification | Independently validate host identity before changing key |
| Authentication failed | User authentication | Username, password/key, authorized login method |
| `Permission denied` for remote root | Filesystem setup | Ownership, file modes, and writable directory |
| `UnsupportedClassVersionError` | Java startup | Runtime version actually selected by the launcher |
| Java executable missing, exit 127 | Java startup | Absolute Java Path or symlink target |
| `remoting.jar` transfer error | File transfer | SFTP access, disk space, directory permissions |
| Agent unexpectedly disconnected | Remoting/runtime | Agent logs, memory, host network, process termination |

**Diagnostic rule:** Read the *last meaningful error after the most recent launch attempt*. Old log failures can remain visible even after a later successful connection.

## 10. Security and operations considerations

- Give agents only the privileges they need; builds can access the agent account's files and network.
- Limit credential scope, rotate keys/passwords, and avoid credentials in job scripts or source control.
- Protect Jenkins home, SSH private keys, credentials, and build workspaces.
- Verify host fingerprints instead of disabling SSH verification.
- Consider agent lifecycle management, patching, filesystem cleanup, monitoring, and resource isolation.
- Executors affect concurrency; more executors do not necessarily mean a host can safely run more concurrent tasks.
- Use a production-grade secrets and identity strategy rather than assuming temporary-lab password credentials are ideal long term.

## 11. Interview questions and answers

**Q1. Why use Jenkins agents?**  
To distribute builds, support distinct runtime environments, enable concurrency, and reduce execution load on the controller.

**Q2. How is a label different from a node name?**  
A node name identifies one node; a label is a scheduler selection tag that may match one or many nodes.

**Q3. Why would SSH succeed yet the node remain offline?**  
Because Java may be missing or incompatible, the remote root may be unwritable, `remoting.jar` may fail to start, or the communication channel may not establish.

**Q4. Does installing Java 21 change `java -version` automatically?**  
Not necessarily. The default might remain selected through alternatives or `PATH`.

**Q5. What does `UnsupportedClassVersionError` mean?**  
The bytecode was compiled for a newer Java version than the runtime attempting to execute it.

**Q6. What does exit code `127` indicate in the launcher log?**  
The shell could not find or execute the configured command, often due to a wrong Java executable path.

**Q7. What proves the agent is online?**  
A successfully established Remoting connection, confirmed by the agent log and current Nodes status; merely configuring the node is insufficient.

**Q8. What is the difference between `known_hosts` and `authorized_keys`?**  
`known_hosts` lets the SSH client verify remote host identity; `authorized_keys` on a server lists public keys allowed to authenticate users there.

## 12. Final outcome

All three required SSH agents were reported online, and **KodeKloud Day 75 passed**. The strongest learning came from separating SSH host trust, remote login authentication, Java runtime compatibility, executable path resolution, and Jenkins Remoting startup into distinct diagnostic stages.
