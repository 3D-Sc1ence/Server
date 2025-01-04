class CloudVariableExtension {
    constructor(runtime) {
        this.runtime = runtime;
    }

    getInfo() {
        return {
            id: 'cloudVariables',
            name: 'Cloud Variables',
            blocks: [
                {
                    opcode: 'getCloudVariable',
                    blockType: 'reporter',
                    text: 'get cloud variable [VARIABLE]',
                    arguments: {
                        VARIABLE: { type: 'string', defaultValue: 'score' }
                    }
                },
                {
                    opcode: 'setCloudVariable',
                    blockType: 'command',
                    text: 'set cloud variable [VARIABLE] to [VALUE]',
                    arguments: {
                        VARIABLE: { type: 'string', defaultValue: 'score' },
                        VALUE: { type: 'number', defaultValue: 0 }
                    }
                }
            ]
        };
    }

    // Block for getting cloud variable
    async getCloudVariable(args) {
        const variable = args.VARIABLE;
        try {
            const response = await fetch(`http://localhost:5000/get/${variable}`);
            if (!response.ok) {
                throw new Error(`Failed to fetch variable: ${response.statusText}`);
            }
            const data = await response.json();
            return data.value;  // Returns the value of the variable
        } catch (error) {
            console.error('Error fetching cloud variable:', error);
            return null;  // Return null in case of error
        }
    }

    // Block for setting cloud variable
    async setCloudVariable(args) {
        const variable = args.VARIABLE;
        const value = args.VALUE;
        const payload = JSON.stringify({ variable, value });

        try {
            const response = await fetch('http://localhost:5000/set', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: payload
            });

            if (!response.ok) {
                throw new Error(`Failed to set variable: ${response.statusText}`);
            }

            const data = await response.json();
            return data.success;  // Return success status
        } catch (error) {
            console.error('Error setting cloud variable:', error);
            return false;  // Return false in case of error
        }
    }
}

// Register the extension
Scratch.extensions.register(new CloudVariableExtension());
