    func TestGroupedParallel(t *testing.T) {
        for _, tc := range tests {
            tc := tc // capture range variable
            t.Run(tc.Name, func(t *testing.T) {
                t.Parallel()
                ...
            })
        }
    }