# pesieve-go
Golang bindings for [PE-sieve](https://github.com/hasherezade/pe-sieve/).

Requires pe-sieve32.dll, pe-sieve64.dll (of a supported version) to be present in the project directory, or in a path pointed by the environment variable: `PESIEVE_DIR`.

## API

Exposes the following wrappers for [PE-sieve API](https://github.com/hasherezade/pe-sieve/wiki/5.-API):

```go
PESieveVersion uint32
func PESieveHelp()
func PESieveScan(pp PEsieveParams) PEsieveReport 
func PESieveScanEx(pp PEsieveParams, rtype t_report_type, jsonMaxSize uint32) (PEsieveReport, string, uint32)
```

## Demo

```go
package main

import (
	"fmt"
	"syscall"
	"github.com/hasherezade/pesieve-go"
)

// Scan the current process
func ScanThis(myPid uint32) string {

	var mods = string("kernel32.dll")
	ignoredBuf := make([]byte, len(mods)+1)
	copy(ignoredBuf[:], mods)

	// Set up the scan parameters
	pp := pesieve.PEsieveParams{
		Pid:      myPid,
		Threads:  true,
		Shellcode: true,
		Quiet:    true,
		JsonLvl: pesieve.JSON_DETAILS2,
		ModulesIgnored : pesieve.PARAM_STRING { Buffer: ignoredBuf, Length: uint32(len(mods)) },
	}
	copy(pp.OutputDir[:], "MyDemoDir")

	const rtype = pesieve.REPORT_ALL
	const jsonMaxLen = 2000
	_, json, _ := pesieve.PESieveScanEx(pp, rtype, jsonMaxLen)
	return json
}


func main() {
	message := ScanThis( uint32(syscall.Getpid()) )
	fmt.Println(message)
}
```
