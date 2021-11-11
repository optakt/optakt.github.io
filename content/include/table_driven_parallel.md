    func TestParseCadenceArgument(t *testing.T) {
        tests := []struct {
            name     string
            param    string
            wantArg  cadence.Value
            checkErr assert.ErrorAssertionFunc
        }{
            {
                name:     "parse valid boolean",
                param:    "Bool(true)",
                wantArg:  cadence.Bool(true),
                checkErr: assert.NoError,
            },
            {
                name:     "parse invalid boolean",
                param:    "Bool(horse)",
                checkErr: assert.Error,
            },
            {
                name:     "parse valid normal integer",
                param:    "Int16(1337)",
                wantArg:  cadence.Int16(1337),
                checkErr: assert.NoError,
            },
        }
    
        for _, test := range tests {
            test := test // It is essential to capture the loop variable, or each parallel test will run with the inputs of the last test from the table.
            t.Run(test.name, func(t *testing.T) {
                t.Parallel()
    
                gotArg, err := ParseCadenceArgument(test.param)
                test.checkErr(t, err)
    
                if err == nil {
                    assert.Equal(t, test.wantArg, gotArg)
                }
            })
        }
    }