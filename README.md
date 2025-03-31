# Prometheus & Grafana on EC2 with Helm and Minikube  

Deploy Prometheus and Grafana on an AWS EC2 Linux instance using Minikube and Helm.  

---

## **Prerequisites**  
1. **AWS EC2 Instance**:   
   - Security group with inbound rules:  
     - SSH (22), Prometheus (30090), Grafana (30100).  

2. **Tools Installed**:  
   - `kubectl`, `helm`, `minikube`, `aws-cli`, `ufw`.  

---

## **Setup Steps**  

### **1. Start Minikube**  
```bash
minikube start --driver=docker
```
### **2. Deploy Prometheus** 
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus \
  --set server.service.type=NodePort \
  --set server.service.nodePort=30090 \
  --set alertmanager.service.type=NodePort
```
### **3. Deploy Grafana** 
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana \
  --set service.type=NodePort \
  --set service.nodePort=30100
```
### **4. Configure EC2 Networking**
```bash
# IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# Set up iptables port forwarding
sudo iptables -t nat -A PREROUTING -p tcp --dport 30090 -j DNAT --to-destination 192.168.49.2:30090
sudo iptables -t nat -A PREROUTING -p tcp --dport 30100 -j DNAT --to-destination 192.168.49.2:30100
sudo iptables -A FORWARD -p tcp --dport 30090 -j ACCEPT
sudo iptables -A FORWARD -p tcp --dport 30100 -j ACCEPT

# Save iptables rules
sudo apt-get install -y iptables-persistent
sudo netfilter-persistent save

# Enable UFW firewall
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 30090/tcp # Prometheus
sudo ufw allow 30100/tcp # Grafana
sudo ufw enable
```
### **5. Access Services**
- External Access
- Prometheus :
  ```bash
  http://<EC2-Public-IP>:30090
  ``` 
- Grafana :
```bash
 http://<EC2-Public-IP>:30100
```
