Ganglia’s an open source, scalable distributed monitoring system for high-performance computing system such as clusters or grid. 
It is based on a hierarchical design, designed for federations of clusters. It leverages widely used technologies such as XML for data # # representation, XDR for compact, portable data transport, and RRDtool for data storage and visualisation. It uses carefully engineered data structures & algorithms to achieve very low per-node overheads and high concurrency. This implementation is robust and has been ported to an extensive set of operating systems and processor architectures.


Download epel-release rpm file on both the Client node and the monitored node. Enter as root to download and install the epel file

- #cd ~/Desktop
- #wget https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-8.noarch.rpm
- #rpm –ivh epel-release-7-8.noarch.rpm

- #yum install ganglia rrdtool ganglia-gmetad ganglia-gmond ganglia-web
THE COMMAND ABOVE IS FOR CLIENT NODE

- #yum install ganglia ganglia-gmond
THE COMMAND ABOVE IS FOR MONITORED NODE

Configuration of Ganglia gmetad.conf is  ONLY for the client node. Enter to the gmetad.conf by entering command below.
- #vi /etc/ganglia/gmetad.conf
Once we enter the gmetad.conf file, we will edit the data source and grid name.

- data_source “Hadoop_Cluster” 60 //insert IP Address of Client Node here//
- gridname “Hadoop Cluster”

Once we had successfully configured the Client node, we will enter these commands to start our ganglia.
- #systemctl restart gmetad gmond httpd

Configuration of Ganglia gmond.conf for all nodes. Enter to the gmond.conf by entering the command below.
- #vi /etc/ganglia/gmond.conf

After which, look out for CLuster, udp send/receive and tcp accept channel. 

Cluster {

   Name = “Hadoop_Cluster”
   
   Ownder = “Me”   
   
   Latlong = “unspecified”
   
   url = “unspecified”
   
}


udp_send_channel {

  host = IP address of the main monitoring system
  
  port = 8694
  
  ttl = 1
  
}


udp_recv_channel {

port = 8694

}


tcp_accept_channel {

port = 8694

}


Once you’ve enter the configurations correctly, save the gmond.conf and restart gmond service. Please allow the connection to take several minutes to load the website as data are being transmitted to and fro between client and monitored nodes.

- #systemctl restart gmond
Launching the web.	We can enter the webpage by launching the web browser and enter the Client’s IP address with “ /ganglia”
Example: 172.16.241.224/ganglia.

To allow Ganglia WebUI to be accessible to all computers apart from the Linux VM, enter and edit ganglia.conf file.

 - #vi /etc/httpd/conf.d/ganglia.conf
 
Add in these lines in <Location /ganglia> properties
- Allow from all
- Require all granted
