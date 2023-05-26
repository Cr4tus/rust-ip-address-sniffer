# Rust Ip Address Sniffer

## What does the app do?
The app will scan the IP addres specified by the user as a program argument. It also accepts 2 flags:
- *-h / --help* (shows a helpful message)
- *-t / --threads-no* + [natural number] (lets you to manually set how many threads will be used)

## Programming Languages:
- Rust

## Frameworks & Libraries:
- rand - https://crates.io/crates/rand | https://docs.rs/rand/0.8.5/rand/

## Implementation:
First, we create a utility **struct**: *IpSnifferArguments*:
```rust
struct IpSnifferArguments {
    threads_no: u16,
    ip_addr: IpAddr,
}
```
and an implementation for the *new* method for it. The most important is the *scan_ip_addr* function:
```rust
n scan_ip_addr(ip_addr: IpAddr, sender: Sender<u16>, starting_port: u16, threads_no: u16) {
    let mut port: u16 = starting_port + 1;
    loop {
        if let Ok(sock_addr) = SocketAddr::try_from((ip_addr, port)) {
            match TcpStream::connect_timeout(
                &sock_addr,
                Duration::from_secs(TCP_CONNECTION_TRIAL_TIMEOUT_SECS as u64)
            ) {
                Ok(_) => {
                    print!(".");

                    io::stdout().flush().unwrap();
                    sender.send(port).unwrap();
                }
                Err(_) => {}
            }
        }

        if threads_no >= MAX_PORT_NO - port {
            break;
        }

        port += threads_no;
    }
}
```
Here, we try to establish a TCP connection. If we succeed it, the port must be opened and we report this through the *sender* that we pass to this function. Each thread will call this function with the correct parameters to split the ports in such manner that a port won't be check twice by one or more different threads.
```rust
for i in 0..ip_sniffer_args.threads_no {
    // cloning the transmitter, so each thread will have its owned transmitter
    let transmitter_clone = sender.clone();

    thread::spawn(move || {
        scan_ip_addr(ip_addr, transmitter_clone, i, threads_no);
    });
}
```
