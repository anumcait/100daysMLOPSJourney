# Day 30 - Create a Health Monitor Script for an ML Application

## Objective
Create a custom health monitor script that checks a local service endpoint and internal logic, returning appropriate exit codes.

## Task
1. Create a health endpoint in the application.
2. Create the monitor script (`monitor.sh`).
3. Make it executable.
4. Test the monitor.
5. Final verification.

## Solution

### Run these commands on the controlplane host:

#### Step 1: Create a health endpoint in the application
Assuming a simple Python/FastAPI app already exists on port 5001.

```python
@app.get("/health")
def health_check():
    return {"status": "ok"}
```

#### Step 2: Create the monitor script (/root/code/monitor.sh)

```bash
cat << 'EOF' > /root/code/monitor.sh
#!/bin/bash
STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:5001/health)

if [ "$STATUS" -eq 200 ]; then
  echo "healthy"
  exit 0
fi

echo "unhealthy"
exit 1
EOF
```

#### Step 3: Make it executable:

```bash
chmod +x /root/code/monitor.sh
```

#### Step 4: Test the monitor

```bash
/root/code/monitor.sh
echo $?
```

**Expected output:**
```
healthy
0
```

#### Step 5: Final verification

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:5001/health

/root/code/monitor.sh
echo $?
```

**Expected output:**
```
200
healthy
0
```

### Summary
Developing a custom health monitor script ensures that we can programmatically verify the health of our services. By checking specific endpoints and returning standard exit codes, this script can be easily integrated into automated monitoring systems or CI/CD pipelines to ensure service reliability.

### Screenshots
<img width="1575" height="778" alt="image" src="https://github.com/user-attachments/assets/328b292b-da8a-47ae-b4c9-5180f9f60e9b" />
<img width="1588" height="771" alt="image" src="https://github.com/user-attachments/assets/e01044f4-3094-46f5-be3a-a19addf7610d" />
