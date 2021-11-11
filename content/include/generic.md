    func GenericAddresses(number int) []flow.Address {
    // Ensure consistent deterministic results.
    random := rand.New(rand.NewSource(5))
    
        var addresses []flow.Address
        for i := 0; i < number; i++ {
            var address flow.Address
            binary.BigEndian.PutUint64(address[0:], random.Uint64())
    
            addresses = append(addresses, address)
        }
    
        return addresses
    }
    
    func GenericAddress(index int) flow.Address {
    return GenericAddresses(index + 1)[index]
    }