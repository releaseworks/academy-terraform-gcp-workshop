# Terraform on GCP
Welcome to the Terraform on GCP workshop. This repository provides example files for the tasks in this workshop.

## Prerequisites
- Get access to a Google Cloud project
- Check that the Compute Engine API is enabled

## Your first Terraform configuration
Start by creating a `main.tf` with the following resources defined:
### Network
```
resource "google_compute_network" "vpc_network" {
  name                    = "workshop-network"
  auto_create_subnetworks = false
}
```

### Subnetwork
```
resource "google_compute_subnetwork" "default" {
  name          = "workshop-subnet"
  ip_cidr_range = "10.0.1.0/24"
  region        = "us-west1"
  network       = google_compute_network.vpc_network.id
}
```

### Google Compute Engine instance (virtual machine)
```
resource "google_compute_instance" "default" {
  name         = "vm"
  machine_type = "f1-micro"
  zone         = "us-west1-a"
  tags         = ["ssh"]

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }

  # Install nginx
  metadata_startup_script = "sudo apt-get update; sudo apt-get install -yq nginx"

  network_interface {
    subnetwork = google_compute_subnetwork.default.id

    access_config {
      # Include this section to get an external IP address for this VM
    }
  }

}
```

### Firewall rule to open port 80 to the world
```
resource "google_compute_firewall" "vm" {
  name    = "vm-firewall"
  network = google_compute_network.vpc_network.id

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }
  source_ranges = ["0.0.0.0/0"]
}
```

### Command cheatsheet
- Initialize Terraform: `terraform init`
- Create a Terraform plan: `terraform plan -out plan.out`
- Apply the plan: `terraform apply plan.out`
- Validate Terraform configuration: `terraform validate`
- Format Terraform configuration: `terraform fmt -recursive` or `terraform fmt -recursive -check` for a pipeline
