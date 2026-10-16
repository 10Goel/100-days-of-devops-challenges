# Day 75 — Jenkins Distributed Builds with SSH Agent Nodes

![Status](https://img.shields.io/badge/KodeKloud-Day%2075%20Passed-2ea44f) ![Jenkins](https://img.shields.io/badge/Jenkins-SSH%20Build%20Agents-2671be) ![Topic](https://img.shields.io/badge/Topic-Distributed%20CI%2FCD-555)

## Overview

Configured three Nautilus application servers as permanent Jenkins SSH build agents, each with its required node name, label, remote working directory, and SSH launcher. Resolved multiple launch failures by validating SSH host keys, matching the Jenkins Remoting Java requirements, and replacing version-dependent Java paths with verified per-user executables.

**Outcome:** KodeKloud Day 75 passed. All three agents were reported online, and the Jenkins logs showed **`Agent successfully connected and online`**.

## Task requirements

| Jenkins node | SSH host | SSH login | Label | Remote root directory |
|---|---|---|---|---|
| `App_server_1` | `stapp01` | `tony` | `stapp01` | `/home/tony/jenkins` |
| `App_server_2` | `stapp02` | `steve` | `stapp02` | `/home/steve/jenkins` |
| `App_server_3` | `stapp03` | `banner` | `stapp03` | `/home/banner/jenkins` |

Additional requirements: use SSH agent launchers; ensure every node is online and functional.

## Architecture

```text
                        Jenkins Controller
                         (Jenkins 2.541.2)
                                |
                  SSH Build Agents plugin
                                |
             +------------------+------------------+
             |                  |                  |
             v                  v                  v
       App_server_1        App_server_2        App_server_3
        stapp01             stapp02             stapp03
        user: tony          user: steve         user: banner
        label: stapp01      label: stapp02       label: stapp03
        /home/tony/jenkins  /home/steve/jenkins /home/banner/jenkins
             |                  |                  |
         Java 21             Java 21             Java 21
         remoting.jar        remoting.jar        remoting.jar
```

The controller manages build scheduling and dispatches work to remote agents. Each agent starts Jenkins Remoting from its configured remote root directory.

## Implementation summary

1. Checked that **SSH Build Agents** (`ssh-slaves`) was installed and enabled in **Manage Jenkins → Plugins**.
2. On each app server, installed or verified OpenJDK 21 and created the required writable Jenkins working directory.
3. Identified the **actual Java 21 executable on each server** and created a verified link such as `/home/tony/java21`; this avoids assuming a package-version-specific path.
4. From the Jenkins Linux account, accepted *verified* SSH server host keys and confirmed entries in `/var/lib/jenkins/.ssh/known_hosts`.
5. Stored each app server's SSH username/password as a separate Jenkins credential; secrets were not embedded in job scripts.
6. Created three **Permanent Agent** nodes with the exact names, labels, directories, SSH hosts, and matching credentials from the table above.
7. Under **Launch agents via SSH → Advanced**, configured the correct per-host Java Path, such as `/home/tony/java21`.
8. Launched each node and checked the respective **Log** for `Agent successfully connected and online`.
9. Verified the live node pages; **Disconnect** was available and an agent executor displayed **0/1** (idle and available).

For the full reproducible procedure and verification commands, see [Commands.md](Commands.md). For concepts and a root-cause troubleshooting guide, see [Notes.md](Notes.md).

## Actual troubleshooting and fixes

| Observed error | Interpretation | Resolution |
|---|---|---|
| `No Known Hosts file was found` | Controller-side SSH host trust was not configured | Create a `known_hosts` file for the `jenkins` Linux account; add verified host keys |
| `UnsupportedClassVersionError` (class 61 vs 55) | Remoting required Java 17 bytecode support but Java 11 was selected | Install/verify Java 21 and configure agent-specific Java Path |
| `No such file or directory` for a Java 21 binary, exit `127` | A previously used Java path did not exist in the current lab | Discover Java's real path; use a tested stable symlink |
| Agent offline after creation | The UI only confirms that the node is registered, not that the remoting process started | Inspect **Node → Log** and address the specific launcher error |

**Key distinction:** An SSH connection and successful authentication are not enough. Jenkins still needs to upload/verify `remoting.jar`, launch the correct Java runtime, and establish the Remoting channel.

## Verification evidence

The successful `App_server_3` log included:

```text
[SSH] Starting agent process: cd "/home/banner/jenkins" && /home/banner/java21 -jar remoting.jar ...
<===[JENKINS REMOTING CAPACITY]===>channel started
Launcher: SSHLauncher
Communication Protocol: Standard in/out
This is a Unix agent
Agent successfully connected and online
```

The same successful connection outcome was reported for the other two agents. The user subsequently confirmed **Day 75 passed** in KodeKloud.

## Security and operational considerations

- Verify SSH host keys using a trusted channel; `ssh-keyscan` alone does not authenticate a server.
- Store credentials in Jenkins Credentials rather than exposing passwords in scripts, logs, or repositories.
- Prefer agent-specific Java paths over changing the host's global Java alternative.
- Limit agent account permissions and protect workspaces and files; builds may execute untrusted code.
- For production environments, use controlled credentials, network restrictions, agent monitoring, and a supported Java/Jenkins version combination.
- Never upload private SSH keys, passwords, `known_hosts` with internal inventory details, or live agent logs containing secrets into a public GitHub repository.

## Learning outcomes

- Jenkins controller versus agent responsibilities
- Permanent nodes, executors, labels, and remote workspaces
- SSH host verification versus SSH user authentication
- Jenkins Remoting and Java class-file compatibility
- Correct diagnosis of SSH launcher failures using agent logs
- Safe multi-host automation with host-specific configuration
